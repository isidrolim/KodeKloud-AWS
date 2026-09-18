# AWS Level 2 – Task 020: Configure NAT Gateway for Internet Access in a Private VPC

## Scenario
The Nautilus DevOps team needs to provide outbound internet access to an EC2 instance located in a private subnet.

The EC2 instance must remain private and use a NAT Gateway for outbound connectivity so that an existing cron job can upload a test file to an S3 bucket.

## Requirement

- **VPC:** `xfusion-priv-vpc`
- **Private Subnet:** `xfusion-priv-subnet`
- **Private EC2:** `xfusion-priv-ec2`
- **Public Subnet:** `xfusion-pub-subnet`
- **Public Route Table:** `xfusion-pub-rt`
- **Private Route Table:** `xfusion-priv-rt`
- **NAT Gateway:** `xfusion-natgw`
- **S3 Bucket:** `xfusion-nat-682119540`
- Private EC2 must access the internet through the NAT Gateway
- Do not expose the private EC2 directly to the internet

## Initial State

The following resources already existed:

```text
xfusion-priv-vpc
    └── xfusion-priv-subnet
            └── xfusion-priv-ec2
```

The private EC2 instance did not have a working outbound internet path.

## Troubleshooting Path

The required dependency path was:

```text
xfusion-priv-ec2
        ↓
xfusion-priv-subnet
        ↓
xfusion-priv-rt
        ↓
NAT Gateway
        ↓
xfusion-pub-subnet
        ↓
xfusion-pub-rt
        ↓
Internet Gateway
        ↓
Internet / Amazon S3
```

## Verification Before Fix

Confirmed:

- `xfusion-priv-ec2` was running
- It was located in `xfusion-priv-subnet`
- The subnet belonged to `xfusion-priv-vpc`
- The instance remained private

## Systematic Elimination

### 1. Create the Public Subnet

Created:

```text
xfusion-pub-subnet
CIDR: 10.1.2.0/24
```

The subnet did not overlap with the existing private subnet:

```text
Private subnet: 10.1.1.0/24
Public subnet:  10.1.2.0/24
```

### 2. Create and Attach an Internet Gateway

Created an Internet Gateway and attached it to:

```text
xfusion-priv-vpc
```

### 3. Create the Public Route Table

Created:

```text
xfusion-pub-rt
```

Associated it with:

```text
xfusion-pub-subnet
```

Added:

```text
0.0.0.0/0 → Internet Gateway
```

This made the subnet suitable for hosting the NAT Gateway.

### 4. Allocate an Elastic IP

Allocated an Elastic IP for use by the public NAT Gateway.

### 5. Create the NAT Gateway

Created:

```text
xfusion-natgw
```

Configuration:

```text
Availability mode: Zonal
Subnet:            xfusion-pub-subnet
Connectivity:      Public
Elastic IP:        Allocated EIP
```

Waited until the NAT Gateway status became:

```text
Available
```

### 6. Create the Private Route Table

Created a dedicated route table:

```text
xfusion-priv-rt
```

The VPC main route table was not reused.

Associated:

```text
xfusion-priv-subnet
```

with:

```text
xfusion-priv-rt
```

### 7. Add the NAT Route

Added:

```text
Destination: 0.0.0.0/0
Target:      xfusion-natgw
```

The final routing design became:

```text
PRIVATE SIDE

xfusion-priv-ec2
        ↓
xfusion-priv-subnet
        ↓
xfusion-priv-rt
        ↓
0.0.0.0/0 → xfusion-natgw


PUBLIC SIDE

xfusion-natgw
        ↓
xfusion-pub-subnet
        ↓
xfusion-pub-rt
        ↓
0.0.0.0/0 → Internet Gateway
        ↓
Internet
```

## First Finding

The private EC2 required an outbound internet path without receiving a public IP address.

The correct architecture was therefore:

```text
Private EC2 → NAT Gateway → Internet Gateway
```

rather than exposing the EC2 directly through the Internet Gateway.

## Fix

The required public-side networking components were created:

```text
Public Subnet
→ Public Route Table
→ Internet Gateway
→ Elastic IP
→ NAT Gateway
```

The private subnet was then associated with a dedicated route table containing:

```text
0.0.0.0/0 → xfusion-natgw
```

## Validation

The original success criterion was tested by checking the S3 bucket:

```bash
aws s3 ls s3://xfusion-nat-682119540/
```

Result:

```text
2026-09-18 14:36:06          0 xfusion-test.txt
```

The presence of `xfusion-test.txt` proves that the cron job running on `xfusion-priv-ec2` successfully reached Amazon S3 through the NAT Gateway.

Final validated path:

```text
xfusion-priv-ec2
        ↓
xfusion-priv-subnet
        ↓
xfusion-priv-rt
        ↓
xfusion-natgw
        ↓
xfusion-pub-subnet
        ↓
xfusion-pub-rt
        ↓
Internet Gateway
        ↓
Amazon S3
        ↓
xfusion-test.txt ✅
```

## Lessons Learned

- A private EC2 instance does not need a public IP to access the internet.
- A NAT Gateway provides outbound internet access while keeping the EC2 instance private.
- A public NAT Gateway must reside in a public subnet.
- The public subnet must have a default route to an Internet Gateway.
- The private subnet must have a default route to the NAT Gateway.
- Public and private subnets should use separate route tables when their routing requirements differ.
- Always validate using the actual business requirement rather than only checking resource states.

## Engineering Insight

A NAT Gateway is not automatically useful just because it exists.

The full dependency path must be complete:

```text
Private Instance
→ Private Route Table
→ NAT Gateway
→ Public Subnet
→ Public Route Table
→ Internet Gateway
→ Internet
```

If any one of these components is missing or incorrectly associated, the private instance will not have outbound connectivity.

The best validation was not simply checking that the NAT Gateway showed `Available`, but proving that the private EC2 could perform its intended task by successfully uploading `xfusion-test.txt` to S3.

## Knowledge Check

1. Why must a public NAT Gateway be placed in a public subnet?
2. Why does the private EC2 instance not require a public IP?
3. What route must exist in the private subnet route table?
4. What route must exist in the public subnet route table?
5. Why is an Elastic IP required for a public NAT Gateway?
6. Why was the S3 upload a stronger validation than simply checking the NAT Gateway status?

## Result

The private EC2 instance `xfusion-priv-ec2` successfully gained outbound internet access through the NAT Gateway while remaining isolated from direct inbound internet access.

The cron job successfully uploaded:

```text
xfusion-test.txt
```

to:

```text
xfusion-nat-682119540
```

**Task Status:** ✅ Completed
