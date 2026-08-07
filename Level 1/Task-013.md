# Create AMI from EC2 Instance via AWS CLI

## Problem Statement

Create an **Amazon Machine Image (AMI)** from an existing EC2 instance with the following requirements:

- **Source Instance:** `nautilus-ec2`
- **AMI Name:** `nautilus-ec2-ami`
- **Required AMI State:** `available`
- **Region:** `us-east-1`

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

4. Locate the source EC2 instance.
   ```bash
   aws ec2 describe-instances \
     --filters "Name=tag:Name,Values=nautilus-ec2" \
     --query "Reservations[*].Instances[*].[InstanceId,State.Name,Placement.AvailabilityZone]" \
     --output table \
     --region us-east-1
   ```

5. Check whether the AMI already exists.
   ```bash
   aws ec2 describe-images \
     --owners self \
     --filters "Name=name,Values=nautilus-ec2-ami" \
     --query "Images[*].[ImageId,Name,State,CreationDate]" \
     --output table \
     --region us-east-1
   ```

6. Create the AMI from the EC2 instance.
   ```bash
   aws ec2 create-image \
     --instance-id <instance-id> \
     --name nautilus-ec2-ami \
     --description "AMI created from nautilus-ec2" \
     --region us-east-1
   ```

7. Check the initial AMI state.
   ```bash
   aws ec2 describe-images \
     --image-ids <ami-id> \
     --query "Images[0].[ImageId,Name,State,CreationDate]" \
     --output table \
     --region us-east-1
   ```

8. Wait until the AMI becomes available.
   ```bash
   aws ec2 wait image-available \
     --image-ids <ami-id> \
     --region us-east-1
   ```

## Validation

Verify the AMI is in the `available` state.

```bash
aws ec2 describe-images \
  --image-ids <ami-id> \
  --query "Images[0].[ImageId,Name,State,CreationDate]" \
  --output table \
  --region us-east-1
```

Expected state:

```text
available
```

## Important AWS CLI Commands Learned

- `describe-images` – List and inspect AMIs.
- `--owners self` – Show AMIs owned by the current AWS account.
- `create-image` – Create an AMI from an existing EC2 instance.
- `image-available` – AWS CLI waiter that waits until AMI creation completes.
- AMI lifecycle during creation: `pending` → `available`.
