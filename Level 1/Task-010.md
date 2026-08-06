# Attach Elastic IP to EC2 Instance via AWS CLI

## Problem Statement

Attach an **Elastic IP** to the existing EC2 instance with the following requirements:

- **EC2 Instance:** `nautilus-ec2`
- **Elastic IP Name:** `nautilus-ec2-eip`
- **Region:** `us-east-1`

> **Note:** If no Elastic IP exists, allocate a new one, tag it, and then associate it with the EC2 instance.

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

4. Locate the EC2 instance.
   ```bash
   aws ec2 describe-instances \
     --filters "Name=tag:Name,Values=nautilus-ec2" \
     --query "Reservations[*].Instances[*].[InstanceId,State.Name]" \
     --output table \
     --region us-east-1
   ```

5. Check whether the Elastic IP already exists.
   ```bash
   aws ec2 describe-addresses \
     --query "Addresses[*].[AllocationId,PublicIp,AssociationId,InstanceId,Tags]" \
     --output table \
     --region us-east-1
   ```

6. Allocate a new Elastic IP (if none exists).
   ```bash
   aws ec2 allocate-address \
     --domain vpc \
     --tag-specifications 'ResourceType=elastic-ip,Tags=[{Key=Name,Value=nautilus-ec2-eip}]' \
     --region us-east-1
   ```

7. Associate the Elastic IP with the EC2 instance.
   ```bash
   aws ec2 associate-address \
     --instance-id <instance-id> \
     --allocation-id <allocation-id> \
     --region us-east-1
   ```

## Validation

Verify the Elastic IP is associated with the EC2 instance.

```bash
aws ec2 describe-addresses \
  --allocation-ids <allocation-id> \
  --query "Addresses[*].[PublicIp,InstanceId,NetworkInterfaceId,AssociationId]" \
  --output table \
  --region us-east-1
```

## Important AWS CLI Commands Learned

- `describe-addresses` – List Elastic IP addresses and their associations.
- `allocate-address` – Allocate a new Elastic IP.
- `associate-address` – Associate an Elastic IP with an EC2 instance (via its primary ENI).
- `--allocation-id` – Uniquely identifies an Elastic IP.
- `--instance-id` – Specifies the target EC2 instance for the Elastic IP association.
