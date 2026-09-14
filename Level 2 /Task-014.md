# AWS Level 2 – Task 014: Setting Up a Private VPC with an Isolated EC2 Instance

## Scenario
The Nautilus DevOps team needs a private VPC containing an isolated subnet and EC2 instance. The instance must not be publicly accessible and should only accept traffic originating from within the VPC.

## Requirements

- **Region:** `us-east-1`
- **VPC Name:** `nautilus-priv-vpc`
- **Subnet Name:** `nautilus-priv-subnet`
- **EC2 Name:** `nautilus-priv-ec2`
- **Instance Type:** `t2.micro`
- EC2 must remain private with no public IP
- Security group must allow access only from the VPC CIDR
- No Internet Gateway is required

## Steps

### 1. Create the Private VPC

Go to:

`VPC → Your VPCs → Create VPC`

Configure:

- **Resources to create:** VPC only
- **Name:** `nautilus-priv-vpc`
- **IPv4 CIDR:** `10.0.0.0/16`
- **Tenancy:** Default

Click:

`Create VPC`

---

### 2. Create the Private Subnet

Go to:

`VPC → Subnets → Create subnet`

Configure:

- **VPC:** `nautilus-priv-vpc`
- **Subnet name:** `nautilus-priv-subnet`
- **Availability Zone:** Any available AZ
- **IPv4 subnet CIDR:** `10.0.1.0/24`

Click:

`Create subnet`

---

### 3. Verify the Subnet Is Private

Select:

`nautilus-priv-subnet → Actions → Edit subnet settings`

Ensure:

`Enable auto-assign public IPv4 address → Disabled`

The route table should only require the VPC's local route:

```text
Destination    Target
10.0.0.0/16    local
```

Do **not** create an Internet Gateway or add a `0.0.0.0/0` internet route.

---

### 4. Create the Security Group

Go to:

`EC2 → Security Groups → Create security group`

Configure:

- **Security group name:** `nautilus-priv-sg`
- **VPC:** `nautilus-priv-vpc`

Add an inbound rule allowing traffic only from the VPC CIDR:

| Type | Protocol | Port | Source |
|---|---|---|---|
| All traffic | All | All | `10.0.0.0/16` |

Create the security group.

This allows communication from resources inside the VPC while preventing direct external access.

---

### 5. Launch the Private EC2 Instance

Go to:

`EC2 → Instances → Launch instances`

Configure:

- **Name:** `nautilus-priv-ec2`
- **AMI:** Select an appropriate available Linux AMI
- **Instance type:** `t2.micro`

Under **Network settings**:

- **VPC:** `nautilus-priv-vpc`
- **Subnet:** `nautilus-priv-subnet`
- **Auto-assign public IP:** Disable
- **Security group:** `nautilus-priv-sg`

Launch the instance.

---

## Validation

Go to:

`EC2 → Instances → nautilus-priv-ec2`

Verify:

- **State:** Running
- **Instance type:** `t2.micro`
- **VPC:** `nautilus-priv-vpc`
- **Subnet:** `nautilus-priv-subnet`
- **Public IPv4 address:** None
- A private IPv4 address is assigned
- Security group allows inbound access only from `10.0.0.0/16`

Verify the subnet route table does not contain an Internet Gateway route.

The final architecture is:

```text
             nautilus-priv-vpc
                10.0.0.0/16
                     │
                     │ local routing only
                     ▼
           nautilus-priv-subnet
                10.0.1.0/24
                     │
                     ▼
             Security Group
          Source: 10.0.0.0/16
                     │
                     ▼
            nautilus-priv-ec2
                t2.micro
              Private IP only

             Internet
                 ✕
          No direct access
```

## Result

A private VPC named `nautilus-priv-vpc` was created with the isolated subnet `nautilus-priv-subnet`.

The `nautilus-priv-ec2` instance was deployed as a `t2.micro` without a public IP address, and its security group restricts inbound access to resources originating from within the VPC CIDR.

**Task Status:** ✅ Completed
