# AWS Level 2 – Task 013: Establishing Secure Communication Between Public and Private VPCs via VPC Peering

## Scenario
The Nautilus DevOps team needs to establish communication between the default public VPC and an existing private VPC using **VPC Peering**.

The public EC2 instance must be able to communicate with the private EC2 instance across their **private IP addresses**.

## Requirements

- **Public EC2:** `devops-public-ec2`
- **Private VPC:** `devops-private-vpc`
- **Private VPC CIDR:** `10.1.0.0/16`
- **Private Subnet:** `devops-private-subnet`
- **Private Subnet CIDR:** `10.1.1.0/24`
- **Private EC2:** `devops-private-ec2`
- **VPC Peering Name:** `devops-vpc-peering`
- Configure routes in both VPCs
- Allow ICMP from the public/default VPC CIDR to the private EC2
- Add `/root/.ssh/id_rsa.pub` to the public EC2 user's `authorized_keys`
- SSH to the public EC2 from `aws-client`
- Ping the private EC2 from the public EC2

## Steps

### 1. Identify Both VPCs

Go to:

`VPC → Your VPCs`

Identify:

- The **default VPC** containing `devops-public-ec2`
- `devops-private-vpc`

Note the CIDR of the default VPC.

The private VPC should be:

`10.1.0.0/16`

---

### 2. Create the VPC Peering Connection

Go to:

`VPC → Peering connections → Create peering connection`

Configure:

- **Name:** `devops-vpc-peering`
- **Requester VPC:** Default/public VPC
- **Accepter VPC:** `devops-private-vpc`

Click:

`Create peering connection`

Select the new connection and choose:

`Actions → Accept request`

Verify the status becomes:

`Active`

---

### 3. Configure the Public VPC Route

Go to:

`VPC → Route tables`

Identify the route table associated with the subnet containing:

`devops-public-ec2`

Add a route:

| Destination | Target |
|---|---|
| `10.1.0.0/16` | `devops-vpc-peering` |

Save the route.

This tells the public VPC how to reach the private VPC.

---

### 4. Configure the Private VPC Route

Identify the route table associated with:

`devops-private-subnet`

Add a route:

| Destination | Target |
|---|---|
| `<DEFAULT-VPC-CIDR>` | `devops-vpc-peering` |

Save the route.

Both directions are required because VPC peering does not automatically modify route tables.

The routing path should now be:

`Default VPC CIDR ↔ devops-vpc-peering ↔ 10.1.0.0/16`

---

### 5. Allow ICMP to the Private EC2 Instance

Go to:

`EC2 → Instances → devops-private-ec2 → Security`

Open the security group attached to the private EC2 instance.

Add an inbound rule:

- **Type:** All ICMP - IPv4
- **Source:** CIDR of the default/public VPC

Example:

`<DEFAULT-VPC-CIDR>`

Save the rule.

This allows the public EC2 instance to ping the private EC2 instance across the peering connection.

---

### 6. Prepare SSH Access to the Public EC2

On `aws-client`, verify the existing public key:

```bash
cat /root/.ssh/id_rsa.pub
```

Copy the complete public key.

Connect to `devops-public-ec2` using the available access method/key and add the public key to the EC2 user's:

```bash
~/.ssh/authorized_keys
```

Ensure the permissions are correct:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

---

### 7. Test SSH from `aws-client`

Obtain the public IP address of:

`devops-public-ec2`

Then test SSH using:

```bash
ssh -i /root/.ssh/id_rsa <ec2-user>@<PUBLIC-IP>
```

Replace `<ec2-user>` with the appropriate AMI user, such as `ec2-user` or `ubuntu`.

Successful authentication confirms:

`aws-client → Internet → devops-public-ec2`

---

### 8. Identify the Private EC2 IP

From the AWS Console:

`EC2 → Instances → devops-private-ec2`

Copy its **Private IPv4 address**.

Do not use a public IP for the VPC peering test.

---

### 9. Test VPC Peering

While logged into `devops-public-ec2`, ping the private EC2:

```bash
ping -c 4 <PRIVATE-EC2-IP>
```

A successful reply validates the complete path:

`devops-public-ec2 → Public VPC Route Table → VPC Peering → Private VPC Route Table → Private EC2 Security Group → devops-private-ec2`

## Validation

Verify:

- `devops-vpc-peering` is `Active`
- Public VPC route table contains `10.1.0.0/16 → VPC Peering`
- Private VPC route table contains `<DEFAULT-VPC-CIDR> → VPC Peering`
- Private EC2 security group permits ICMP from the default VPC CIDR
- `/root/.ssh/id_rsa.pub` allows SSH access from `aws-client` to `devops-public-ec2`
- SSH to the public EC2 succeeds
- `devops-public-ec2` can ping the **private IP** of `devops-private-ec2`

## Result

Secure private communication was established between the default public VPC and `devops-private-vpc` using VPC Peering.

The final communication path is:

`aws-client → SSH → devops-public-ec2 → VPC Peering → devops-private-ec2`

The public EC2 instance can successfully reach the private EC2 instance using its private IPv4 address without requiring public internet access for the private instance.

**Task Status:** ✅ Completed
