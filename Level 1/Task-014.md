# Terminate EC2 Instance via AWS CLI

## Problem Statement

Terminate the specified **EC2 instance** that is no longer required.

- **Region:** `us-east-1`
- **Final Instance State:** `terminated`

## Steps

1. Verify the AWS CLI is authenticated.
   ```bash
   aws sts get-caller-identity
   ```

2. Verify the AWS CLI configuration and Region.
   ```bash
   aws configure list
   aws configure get region
   ```

3. Locate the target EC2 instance.
   ```bash
   aws ec2 describe-instances \
     --filters "Name=tag:Name,Values=<instance-name>" \
     --query "Reservations[*].Instances[*].[InstanceId,State.Name,InstanceType]" \
     --output table \
     --region us-east-1
   ```

4. Terminate the EC2 instance.
   ```bash
   aws ec2 terminate-instances \
     --instance-ids <instance-id> \
     --region us-east-1
   ```

5. Wait until termination is complete.
   ```bash
   aws ec2 wait instance-terminated \
     --instance-ids <instance-id> \
     --region us-east-1
   ```

## Validation

Verify the final instance state.

```bash
aws ec2 describe-instances \
  --instance-ids <instance-id> \
  --query "Reservations[*].Instances[*].[InstanceId,State.Name]" \
  --output table \
  --region us-east-1
```

Expected state:

```text
terminated
```

## Important AWS CLI Commands Learned

- `terminate-instances` – Permanently terminate an EC2 instance.
- `wait instance-terminated` – Wait until AWS confirms termination is complete.
- An EC2 instance does **not** need to be stopped before termination.
- Always verify the **Instance ID** before terminating because termination is destructive.
