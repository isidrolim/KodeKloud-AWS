# Create Subnet via AWS CLI

## Problem Statement

Create an **Amazon VPC Subnet** with the following requirements:

- **Subnet Name:** `devops-subnet`
- **VPC:** Default VPC
- **Region:** `us-east-1`

> **Note:** A subnet requires an unused CIDR block within the VPC. If the selected CIDR overlaps an existing subnet, choose another available CIDR range.

## Steps

1. Verify the AWS CLI is authenticated.
   ```bash
   aws sts get-caller-identity
   ```

2. Verify the AWS CLI configuration.
   ```bash
   aws configure list
   ```

3. Verify the target Region.
   ```bash
   aws configure get region
   ```

4. Retrieve the default VPC ID.
   ```bash
   aws ec2 describe-vpcs \
     --filters "Name=is-default,Values=true" \
     --query "Vpcs[0].VpcId" \
     --output text \
     --region us-east-1
   ```

5. Check whether the subnet already exists.
   ```bash
   aws ec2 describe-subnets \
     --filters "Name=tag:Name,Values=devops-subnet" \
     --query "Subnets[*].[SubnetId,AvailabilityZone,CidrBlock]" \
     --output table \
     --region us-east-1
   ```

6. Create the subnet.
   ```bash
   aws ec2 create-subnet \
     --vpc-id <vpc-id> \
     --cidr-block 172.31.96.0/20 \
     --availability-zone us-east-1a \
     --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=devops-subnet}]' \
     --region us-east-1
   ```

## Validation

Verify the subnet configuration.

```bash
aws ec2 describe-subnets \
  --filters "Name=tag:Name,Values=devops-subnet" \
  --query "Subnets[*].[SubnetId,VpcId,AvailabilityZone,CidrBlock,State]" \
  --output table \
  --region us-east-1
```
