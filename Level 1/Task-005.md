# Allocate Elastic IP via AWS CLI

## Problem Statement

Allocate an **Elastic IP Address** with the following requirements:

- **Elastic IP Name:** `datacenter-eip`
- **Region:** `us-east-1`

> **Note:** Elastic IPs do not have a native name attribute. The name is assigned using the `Name` tag.

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

4. List existing Elastic IPs.
   ```bash
   aws ec2 describe-addresses \
     --query "Addresses[*].[AllocationId,PublicIp,AssociationId,Tags]" \
     --output table \
     --region us-east-1
   ```

5. Check whether `datacenter-eip` already exists.
   ```bash
   aws ec2 describe-addresses \
     --filters "Name=tag:Name,Values=datacenter-eip" \
     --query "Addresses[*].[AllocationId,PublicIp,AssociationId]" \
     --output table \
     --region us-east-1
   ```

6. Allocate the Elastic IP.
   ```bash
   aws ec2 allocate-address \
     --domain vpc \
     --region us-east-1
   ```

7. Tag the Elastic IP.
   ```bash
   aws ec2 create-tags \
     --resources <allocation-id> \
     --tags Key=Name,Value=datacenter-eip \
     --region us-east-1
   ```

## Validation

Verify the Elastic IP and its `Name` tag.

```bash
aws ec2 describe-addresses \
  --filters "Name=tag:Name,Values=datacenter-eip" \
  --query "Addresses[*].[AllocationId,PublicIp,Domain,Tags]" \
  --output table \
  --region us-east-1
```
