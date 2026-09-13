# AWS Level 2 – Task 012: Configuring a Public VPC with an EC2 Instance for Internet Access

## Scenario
The Nautilus DevOps Team needs a new public VPC for internet-facing services. The VPC must contain a public subnet that automatically assigns public IP addresses, along with an EC2 instance that can be accessed over SSH from the internet.

## Requirements

- **Region:** `us-east-1`
- **VPC Name:** `xfusion-pub-vpc`
- **Subnet Name:** `xfusion-pub-subnet`
- Enable automatic public IPv4 assignment on the subnet
- **EC2 Name:** `xfusion-pub-ec2`
- **Instance Type:** `t2.micro`
- Allow inbound SSH on TCP port `22`
- EC2 instance must have internet connectivity

## Steps

### 1. Create the VPC

Go to:

`AWS Console → VPC → Your VPCs → Create VPC`

Configure:

- **Resources to create:** VPC only
- **Name:** `xfusion-pub-vpc`
- **IPv4 CIDR:** `10.0.0.0/16`
- **IPv6:** No IPv6 CIDR block
- **Tenancy:** Default

Click:

`Create VPC`

---

### 2. Create the Public Subnet

Go to:

`VPC → Subnets → Create subnet`

Configure:

- **VPC:** `xfusion-pub-vpc`
- **Subnet name:** `xfusion-pub-subnet`
- **Availability Zone:** Select an available AZ
- **IPv4 subnet CIDR:** `10.0.1.0/24`

Click:

`Create subnet`

---

### 3. Enable Automatic Public IP Assignment

Select:

`xfusion-pub-subnet`

Go to:

`Actions → Edit subnet settings`

Enable:

`Enable auto-assign public IPv4 address`

Save the changes.

This ensures instances launched in the subnet automatically receive a public IPv4 address.

---

### 4. Create an Internet Gateway

Go to:

`VPC → Internet gateways → Create internet gateway`

Configure:

- **Name:** `xfusion-pub-igw`

Click:

`Create internet gateway`

Then select:

`Actions → Attach to a VPC`

Choose:

`xfusion-pub-vpc`

Attach the Internet Gateway.

---

### 5. Configure the Public Route

Go to:

`VPC → Route tables`

Select the route table associated with `xfusion-pub-vpc`.

If necessary, associate it with:

`xfusion-pub-subnet`

Under **Routes**, select:

`Edit routes → Add route`

Configure:

| Destination | Target |
|---|---|
| `0.0.0.0/0` | `xfusion-pub-igw` |

Save the route.

The subnet now has an internet route:

`xfusion-pub-subnet → 0.0.0.0/0 → Internet Gateway`

---

### 6. Launch the EC2 Instance

Go to:

`EC2 → Instances → Launch instances`

Configure:

- **Name:** `xfusion-pub-ec2`
- **AMI:** Select an appropriate available Linux AMI
- **Instance type:** `t2.micro`
- **VPC:** `xfusion-pub-vpc`
- **Subnet:** `xfusion-pub-subnet`
- **Auto-assign public IP:** Enable

Select or create an appropriate key pair if required.

---

### 7. Configure SSH Access

Create or select a security group for the instance.

Add the following inbound rule:

| Type | Protocol | Port | Source |
|---|---|---|---|
| SSH | TCP | 22 | `0.0.0.0/0` |

Launch the instance.

> Opening SSH to `0.0.0.0/0` satisfies the lab requirement for internet accessibility. In production, restrict SSH to trusted administrative IP addresses or use AWS Systems Manager Session Manager.

---

## Validation

Go to:

`EC2 → Instances → xfusion-pub-ec2`

Confirm:

- **State:** Running
- **Instance type:** `t2.micro`
- **VPC:** `xfusion-pub-vpc`
- **Subnet:** `xfusion-pub-subnet`
- A **Public IPv4 address** was automatically assigned
- Security group allows TCP/22

Verify the subnet:

`VPC → Subnets → xfusion-pub-subnet`

Confirm:

`Auto-assign public IPv4 address: Yes`

Verify the route table contains:

`0.0.0.0/0 → Internet Gateway`

The final network path is:

`Internet → Internet Gateway → Public Route Table → xfusion-pub-subnet → Security Group TCP/22 → xfusion-pub-ec2`

## Result

A public VPC named `xfusion-pub-vpc` was created with the public subnet `xfusion-pub-subnet`.

The subnet was configured to automatically assign public IPv4 addresses and was provided internet connectivity through an Internet Gateway and a default route.

A `t2.micro` EC2 instance named `xfusion-pub-ec2` was launched inside the public subnet with SSH port `22` accessible from the internet.

**Task Status:** ✅ Completed
