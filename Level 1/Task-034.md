# Create a VPC with Specific CIDR via AWS CLI

## Problem Statement

Create a new Amazon VPC with the following requirements:

- **VPC Name:** `xfusion-vpc`
- **IPv4 CIDR:** `192.168.0.0/24`
- **Region:** `us-east-1`

## Initial State

The VPC did not exist and needed to be created using the required CIDR block and Name tag.

## Steps

1. Verify the AWS CLI identity, configuration, and Region.
   ```bash
   aws sts get-caller-identity
   aws configure list
   aws configure get region
   ```

2. Check existing VPCs before making changes.
   ```bash
   aws ec2 describe-vpcs \
     --query "Vpcs[*].[VpcId,CidrBlock,IsDefault,Tags[?Key=='Name']|[0].Value]" \
     --output table \
     --region us-east-1
   ```

3. Create the VPC with the required CIDR block and Name tag.
   ```bash
   aws ec2 create-vpc \
     --cidr-block 192.168.0.0/24 \
     --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=xfusion-vpc}]' \
     --region us-east-1
   ```

## Validation

Verify the newly created VPC using its Name tag.

```bash
aws ec2 describe-vpcs \
  --filters "Name=tag:Name,Values=xfusion-vpc" \
  --query "Vpcs[*].[VpcId,State,CidrBlock,IsDefault,Tags[?Key=='Name']|[0].Value]" \
  --output table \
  --region us-east-1
```

Expected result:

```text
State:     available
CIDR:      192.168.0.0/24
IsDefault: False
Name:      xfusion-vpc
```

## Important AWS CLI Commands Learned

- `aws ec2 describe-vpcs` – Inspect existing VPCs.
- `aws ec2 create-vpc` – Create a new VPC.
- `--cidr-block` – Define the IPv4 network assigned to the VPC.
- `--tag-specifications` – Apply tags during resource creation.
- `--filters "Name=tag:Name,Values=..."` – Locate a resource using its Name tag.
- AWS generates the actual `vpc-xxxxxxxx` resource ID; `xfusion-vpc` is the value of the `Name` tag.

## Engineering Insight

The workflow from Task-033 can be reused:

```text
Verify AWS Context
        ↓
Inspect Existing VPCs
        ↓
Validate CIDR Requirement
        ↓
Create VPC
        ↓
Apply Name Tag
        ↓
Query by Name Tag
        ↓
Validate State + CIDR
```

A technically valid CIDR is not automatically a good production CIDR. In a real environment, verify that `192.168.0.0/24` does not overlap with existing VPCs, on-premises networks, VPNs, peered VPCs, or Transit Gateway routes before deployment.
