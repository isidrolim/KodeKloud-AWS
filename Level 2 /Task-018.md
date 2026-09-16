# AWS Level 2 – Task 018: Troubleshooting Connectivity Issues for Package Installation on EC2

## Scenario
The Nautilus DevOps team is unable to install packages on the EC2 instance `xfusion-ec2` because of a connectivity issue.

The goal is to troubleshoot the network path systematically, identify the root cause, restore internet connectivity, and verify that package installation works again.

## Requirements

- **Region:** `us-east-1`
- **EC2 Instance:** `xfusion-ec2`
- **SSH Private Key:** `/root/.ssh/id_rsa`
- Identify the connectivity failure
- Implement the required fix
- Verify package repositories are reachable
- Verify packages can be installed successfully

## Troubleshooting Path

Follow the dependency path instead of changing resources randomly:

```text
xfusion-ec2
    ↓
Network Interface
    ↓
Security Group / NACL
    ↓
Subnet Route Table
    ↓
Internet Gateway or NAT Gateway
    ↓
DNS
    ↓
Package Repository
```

---

## 1. Inspect the EC2 Network Configuration

Go to:

`EC2 → Instances → xfusion-ec2`

Record:

- Instance state
- Private IPv4 address
- Public IPv4 address, if any
- VPC ID
- Subnet ID
- Security groups

Determine whether the instance is located in a public or private subnet.

---

## 2. Connect to the EC2 Instance

The SSH key is already available on `aws-client`:

```bash
ls -l /root/.ssh/id_rsa
```

Ensure appropriate permissions:

```bash
chmod 600 /root/.ssh/id_rsa
```

SSH to the instance using its reachable address:

```bash
ssh -i /root/.ssh/id_rsa <ec2-user>@<EC2-IP>
```

Use the appropriate username for the instance AMI, such as `ec2-user` or `ubuntu`.

---

## 3. Reproduce the Package Installation Problem

Before changing anything, reproduce the failure.

For Amazon Linux/RHEL-based systems:

```bash
sudo dnf makecache
```

or:

```bash
sudo yum makecache
```

For Ubuntu:

```bash
sudo apt update
```

Record whether the failure indicates:

- Network timeout
- DNS resolution failure
- Repository unreachable
- Routing failure

---

## 4. Test Basic Network Connectivity

Check the instance routing table:

```bash
ip route
```

Test connectivity to an external IP:

```bash
ping -c 4 8.8.8.8
```

Then test DNS resolution:

```bash
getent hosts amazon.com
```

Interpret the results:

```text
External IP fails
→ Investigate routing / internet egress

External IP works but DNS fails
→ Investigate DNS configuration

Both work
→ Investigate package repository configuration
```

---

## 5. Verify the Subnet Route Table

Go to:

`VPC → Subnets → <xfusion-ec2-subnet> → Route table`

Inspect the route table actually associated with the EC2 subnet.

For a public EC2 instance, expect:

```text
<VPC-CIDR>    local
0.0.0.0/0    Internet Gateway
```

For a private EC2 instance requiring outbound internet access, expect:

```text
<VPC-CIDR>    local
0.0.0.0/0    NAT Gateway
```

If the required default route is missing or points to the wrong target, correct the route.

---

## 6. Verify Internet Gateway / NAT Connectivity

If the EC2 instance is intended to be public:

- Confirm an Internet Gateway exists
- Confirm it is attached to the instance's VPC
- Confirm the subnet route table contains `0.0.0.0/0 → Internet Gateway`
- Confirm the EC2 instance has a public IPv4 address

If the EC2 instance is private:

- Confirm the private subnet has a default route to a NAT Gateway
- Confirm the NAT Gateway resides in a public subnet
- Confirm that public subnet routes `0.0.0.0/0` to an Internet Gateway

---

## 7. Verify the Security Group

Go to:

`EC2 → xfusion-ec2 → Security`

Package installation normally requires outbound HTTPS/HTTP connectivity.

Verify the security group allows the necessary outbound traffic.

A typical lab configuration is:

```text
Outbound:
All traffic → 0.0.0.0/0
```

Do not modify inbound rules unless evidence shows they are related to the failure.

---

## 8. Verify the Network ACL

Go to:

`VPC → Subnets → <xfusion-ec2-subnet> → Network ACL`

Confirm the NACL permits the required outbound traffic and corresponding return traffic.

Remember:

**Security groups are stateful, while Network ACLs are stateless.**

Therefore, restrictive NACLs must explicitly permit both directions.

---

## 9. Verify DNS

If IP connectivity works but repository names cannot be resolved, inspect:

```bash
cat /etc/resolv.conf
```

Test:

```bash
getent hosts amazon.com
```

Also verify the VPC DNS settings if necessary:

`VPC → Your VPCs → DNS settings`

Confirm DNS support is enabled.

---

## 10. Validate Package Installation

After fixing the first failed dependency, return to `xfusion-ec2`.

For Amazon Linux/RHEL:

```bash
sudo dnf makecache
```

Then test installation with an appropriate package:

```bash
sudo dnf install -y wget
```

For older systems:

```bash
sudo yum install -y wget
```

For Ubuntu:

```bash
sudo apt update
sudo apt install -y wget
```

Successful repository access and package installation confirm that outbound connectivity has been restored.

## Validation

Verify the original success criteria:

```text
xfusion-ec2
    ↓
Network configuration      ✅
    ↓
Subnet routing             ✅
    ↓
Internet egress            ✅
    ↓
DNS resolution             ✅
    ↓
Package repositories       ✅
    ↓
Package installation       ✅
```

Confirm:

- `xfusion-ec2` is running
- Correct subnet route table is associated
- Required internet egress path exists
- Security group outbound traffic is permitted
- NACL does not block required traffic
- DNS resolution works
- Package repositories are reachable
- Package installation completes successfully

## Lessons Learned

- A package installation failure can be a symptom of a network problem rather than a package-manager problem.
- Always reproduce the failure before making changes.
- Test IP connectivity and DNS separately.
- Inspect the route table actually associated with the EC2 subnet rather than assuming the VPC's main route table is being used.
- Public and private EC2 instances require different internet egress architectures.
- Fix the first failed dependency instead of modifying multiple components simultaneously.

## Engineering Insight

For package repository connectivity problems, troubleshoot from the EC2 instance outward:

```text
Application / Package Manager
        ↓
DNS
        ↓
Operating System Routing
        ↓
Security Group
        ↓
Network ACL
        ↓
Subnet Route Table
        ↓
IGW / NAT
        ↓
Internet
```

The key principle is:

**Prove where connectivity first fails, fix only that layer, and validate again using the original package installation command.**

## Result

The connectivity path for `xfusion-ec2` was systematically investigated, the network configuration responsible for preventing repository access was corrected, and package installation capability was restored.

**Task Status:** ✅ Completed
