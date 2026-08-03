# Create Security Group via AWS CLI

## Problem Statement

Create an **EC2 Security Group** in the **default VPC** with the following requirements:

- **Security Group Name:** `devops-sg`
- **Description:** `Security group for Nautilus App Servers`
- **VPC:** Default VPC
- **Inbound Rule 1:** HTTP (TCP 80) from `0.0.0.0/0`
- **Inbound Rule 2:** SSH (TCP 22) from `0.0.0.0/0`

## Steps

1. Verify the AWS CLI is authenticated.
   ```bash
   aws sts get-caller-identity
   ```

2. Verify the target Region.
   ```bash
   aws configure get region
   ```

3. Retrieve the default VPC ID.
   ```bash
   aws ec2 describe-vpcs \
     --filters "Name=is-default,Values=true" \
     --query "Vpcs[0].VpcId" \
     --output text \
     --region us-east-1
   ```

4. Create the security group.
   ```bash
   aws ec2 create-security-group \
     --group-name devops-sg \
     --description "Security group for Nautilus App Servers" \
     --vpc-id <vpc-id> \
     --region us-east-1
   ```

5. Add the HTTP inbound rule.
   ```bash
   aws ec2 authorize-security-group-ingress \
     --group-name devops-sg \
     --protocol tcp \
     --port 80 \
     --cidr 0.0.0.0/0 \
     --region us-east-1
   ```

6. Add the SSH inbound rule.
   ```bash
   aws ec2 authorize-security-group-ingress \
     --group-name devops-sg \
     --protocol tcp \
     --port 22 \
     --cidr 0.0.0.0/0 \
     --region us-east-1
   ```

## Validation

Verify the security group and inbound rules.

```bash
aws ec2 describe-security-groups \
  --group-names devops-sg \
  --query "SecurityGroups[0].IpPermissions" \
  --output table \
  --region us-east-1
```
