# AWS Level 2 – Task 007: Configuring an EC2 Instance as a Web Server with Nginx

## Scenario
The Nautilus DevOps team needs an EC2 instance configured as a web server running Nginx. The server must be automatically configured during launch using EC2 User Data and be accessible from the internet over HTTP.

## Requirements

- **Region:** `us-east-1`
- **Instance Name:** `nautilus-ec2`
- **AMI:** Any available Ubuntu AMI
- Install Nginx using **User Data**
- Start and enable Nginx
- Allow inbound **HTTP TCP/80** from the internet

## Steps

### 1. Launch the EC2 Instance

Go to:

`EC2 → Instances → Launch instances`

Configure:

- **Name:** `nautilus-ec2`
- **AMI:** Ubuntu
- **Instance type:** Select an appropriate available type

---

### 2. Configure the Security Group

Under **Network settings**, create or select a security group.

Add the following inbound rule:

| Type | Protocol | Port | Source |
|---|---|---|---|
| HTTP | TCP | 80 | Anywhere IPv4 (`0.0.0.0/0`) |

This allows the Nginx website to be accessed from the internet.

---

### 3. Configure User Data

Expand:

`Advanced details → User data`

Enter:

```bash
#!/bin/bash
apt-get update -y
apt-get install -y nginx
systemctl enable nginx
systemctl start nginx
