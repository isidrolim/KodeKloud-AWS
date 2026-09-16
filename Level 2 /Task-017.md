# AWS Level 2 – Task 017: Troubleshooting Internet Accessibility for an EC2-Hosted Application

## Scenario
The Nautilus Development Team deployed an Nginx web application on `datacenter-ec2` inside the public VPC `datacenter-vpc`.

The EC2 security group `datacenter-sg` already allows HTTP traffic on port `80`, but the application is still inaccessible from the internet.

The goal is to troubleshoot the complete network path, identify the first failure, fix it, and verify that Nginx is publicly accessible.

## Requirements

- **VPC:** `datacenter-vpc`
- **EC2:** `datacenter-ec2`
- **Security Group:** `datacenter-sg`
- **Application:** Nginx
- **Protocol:** HTTP
- **Port:** `80`
- Application must be accessible from the internet

## Troubleshooting Path

Follow the dependency path:

```text
Internet
   ↓
Internet Gateway
   ↓
VPC Route Table
   ↓
Public Subnet
   ↓
Network ACL
   ↓
EC2 Security Group :80
   ↓
EC2 Public IPv4
   ↓
Nginx :80
```

Do not assume the security group is the problem simply because the website is unreachable.

---

## 1. Verify the EC2 Instance

Go to:

`EC2 → Instances → datacenter-ec2`

Confirm:

- Instance is `Running`
- Instance belongs to `datacenter-vpc`
- A **Public IPv4 address** is assigned
- Note the subnet used by the instance

---

## 2. Verify the Security Group

Go to:

`datacenter-ec2 → Security`

Verify `datacenter-sg` contains:

| Type | Protocol | Port | Source |
|---|---|---|---|
| HTTP | TCP | 80 | `0.0.0.0/0` |

If this already exists, do not modify it unnecessarily.

---

## 3. Verify the Internet Gateway

Go to:

`VPC → Internet gateways`

Confirm an Internet Gateway exists and is attached to:

`datacenter-vpc`

If the VPC does not have an attached Internet Gateway:

1. Create an Internet Gateway.
2. Attach it to `datacenter-vpc`.

---

## 4. Verify the Subnet Route Table

Go to:

`VPC → Subnets`

Select the subnet containing `datacenter-ec2`.

Open:

`Route table`

A public subnet should have a route similar to:

```text
Destination          Target
<VPC-CIDR>           local
0.0.0.0/0            igw-xxxxxxxx
```

If the default internet route is missing, edit the route table and add:

```text
Destination: 0.0.0.0/0
Target:      Internet Gateway attached to datacenter-vpc
```

Save the route.

This is a critical requirement for a subnet to provide direct IPv4 internet connectivity through an Internet Gateway.

---

## 5. Verify the Network ACL

From the EC2 subnet, open:

`Network ACL`

Verify the NACL permits the required inbound and outbound traffic.

For a default/open lab NACL, this may appear as:

```text
Inbound:
100 | All traffic | 0.0.0.0/0 | ALLOW

Outbound:
100 | All traffic | 0.0.0.0/0 | ALLOW
```

Remember that Network ACLs are **stateless**, so both directions must permit the traffic.

---

## 6. Verify Nginx

If the AWS network path looks correct but the website remains unavailable, connect to `datacenter-ec2` and check:

```bash
systemctl status nginx
```

Verify something is listening on TCP/80:

```bash
sudo ss -lntp | grep ':80'
```

Test Nginx locally:

```bash
curl http://localhost
```

A successful local response proves that the application is serving HTTP independently of the external network path.

---

## 7. Test Public Access

Retrieve the EC2 public IPv4 address and open:

```text
http://<PUBLIC-IP>
```

The Nginx page should load successfully.

The completed traffic path should be:

```text
Internet
   ↓
Internet Gateway
   ↓
0.0.0.0/0 Route
   ↓
Public Subnet
   ↓
Network ACL
   ↓
datacenter-sg TCP/80
   ↓
datacenter-ec2
   ↓
Nginx :80
```

## Validation

Confirm:

- `datacenter-ec2` is running
- EC2 has a public IPv4 address
- `datacenter-vpc` has an attached Internet Gateway
- EC2 subnet route table contains `0.0.0.0/0 → Internet Gateway`
- Network ACL permits the required traffic
- `datacenter-sg` permits TCP/80 from the internet
- Nginx is running and listening on TCP/80
- `curl http://localhost` succeeds
- `http://<PUBLIC-IP>` loads the Nginx page externally

## Lessons Learned

- Opening TCP/80 in a security group alone does not make an EC2 instance publicly accessible.
- Public connectivity depends on several independent networking components working together.
- Always verify the route table actually associated with the EC2 subnet rather than modifying an arbitrary VPC route table.
- A timeout and a connection refusal represent different failure modes and should lead to different troubleshooting paths.
- Testing `curl localhost` helps separate application failures from AWS networking failures.

## Engineering Insight

Troubleshoot internet connectivity as a dependency chain rather than changing configurations randomly:

```text
Public IP
→ Internet Gateway
→ Route Table
→ NACL
→ Security Group
→ OS
→ Application
```

Find the **first layer that does not satisfy the expected state**, fix only that layer, and then retest using the original success criterion.

## Result

The VPC networking and EC2 application path were systematically verified and corrected where necessary, allowing the Nginx application hosted on `datacenter-ec2` to become accessible from the internet over HTTP port `80`.

**Task Status:** ✅ Completed
