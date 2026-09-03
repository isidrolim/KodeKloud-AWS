# AWS Level 2 – Task 5: Setting Up an Application Load Balancer for an EC2 Instance

## Scenario
The Nautilus DevOps team needs an Application Load Balancer in front of an existing EC2 instance running Nginx.

The existing instance is:

`devops-ec2`

The ALB will receive HTTP traffic on port `80` and forward it to Nginx on port `80`.

## Requirements

- Application Load Balancer: `devops-alb`
- Target Group: `devops-tg`
- ALB Security Group: `devops-sg`
- ALB Listener: HTTP `80`
- Target: `devops-ec2` on port `80`
- Region: `us-east-1`

## Steps

### 1. Check the Existing EC2 Instance

Go to:

`EC2 → Instances → devops-ec2`

Note the following:

- VPC ID
- Availability Zone
- Subnet
- Security Group

Confirm that `devops-ec2` is in the `Running` state.

---

### 2. Create the ALB Security Group

Go to:

`EC2 → Security Groups → Create security group`

Configure:

- **Security group name:** `devops-sg`
- **VPC:** Same VPC as `devops-ec2`

Add inbound rule:

| Type | Protocol | Port | Source |
|---|---|---|---|
| HTTP | TCP | 80 | Anywhere IPv4 (`0.0.0.0/0`) |

Create the security group.

---

### 3. Create the Target Group

Go to:

`EC2 → Target Groups → Create target group`

Configure:

- **Target type:** Instances
- **Target group name:** `devops-tg`
- **Protocol:** HTTP
- **Port:** 80
- **VPC:** Same VPC as `devops-ec2`

Continue to **Register targets**.

Select:

`devops-ec2`

Set port:

`80`

Click:

`Include as pending below → Create target group`

---

### 4. Allow ALB Traffic to the EC2 Instance

Open the security group currently attached to `devops-ec2`.

Add an inbound rule:

| Type | Port | Source |
|---|---|---|
| HTTP | 80 | `devops-sg` |

This creates the desired traffic path:

`Internet → devops-alb:80 → devops-ec2:80`

The EC2 instance trusts HTTP traffic from the ALB security group rather than directly from the entire internet.

---

### 5. Create the Application Load Balancer

Go to:

`EC2 → Load Balancers → Create Load Balancer → Application Load Balancer`

Configure:

- **Name:** `devops-alb`
- **Scheme:** Internet-facing
- **IP address type:** IPv4
- **VPC:** Same VPC as `devops-ec2`

Under **Network mapping**, select at least **two Availability Zones/subnets**.

Under **Security groups**, select:

`devops-sg`

Remove the default security group if it was automatically selected and is not required.

---

### 6. Configure the Listener

Configure the listener:

- **Protocol:** HTTP
- **Port:** 80
- **Default action:** Forward to `devops-tg`

Create the load balancer.

---

## Validation

Wait until:

`devops-alb → State: Active`

Then check:

`EC2 → Target Groups → devops-tg → Targets`

The `devops-ec2` target should become:

`Healthy`

Copy the **DNS name** of `devops-alb` and open:

`http://<ALB-DNS-NAME>`

The Nginx sample page should load successfully.

## Result

The traffic path is now:

`Internet → devops-alb:80 → devops-tg → devops-ec2:80 → Nginx`

**Task Status:** ✅ Completed
