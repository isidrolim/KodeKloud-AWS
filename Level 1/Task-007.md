# Change EC2 Instance Type via AWS CLI

## Problem Statement

Modify an existing EC2 instance with the following requirements:

- **Instance Name:** `devops-ec2`
- **Current Type:** `t2.micro`
- **New Type:** `t2.nano`
- **Final State:** `running`
- **Region:** `us-east-1`

> **Note:** Ensure the EC2 status checks are **OK** before modifying the instance type.

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
     --filters "Name=tag:Name,Values=devops-ec2" \
     --query "Reservations[*].Instances[*].[InstanceId,State.Name,InstanceType]" \
     --output table \
     --region us-east-1
   ```

5. Verify the EC2 status checks.
   ```bash
   aws ec2 describe-instance-status \
     --include-all-instances \
     --instance-ids <instance-id> \
     --query "InstanceStatuses[*].[InstanceStatus.Status,SystemStatus.Status]" \
     --output table \
     --region us-east-1
   ```

6. Stop the instance.
   ```bash
   aws ec2 stop-instances \
     --instance-ids <instance-id> \
     --region us-east-1
   ```

7. Wait until the instance is fully stopped.
   ```bash
   aws ec2 wait instance-stopped \
     --instance-ids <instance-id> \
     --region us-east-1
   ```

8. Change the instance type.
   ```bash
   aws ec2 modify-instance-attribute \
     --instance-id <instance-id> \
     --instance-type "{\"Value\":\"t2.nano\"}" \
     --region us-east-1
   ```

9. Start the instance.
   ```bash
   aws ec2 start-instances \
     --instance-ids <instance-id> \
     --region us-east-1
   ```

10. Wait until the instance is running.
    ```bash
    aws ec2 wait instance-running \
      --instance-ids <instance-id> \
      --region us-east-1
    ```

## Validation

Verify the instance state and new instance type.

```bash
aws ec2 describe-instances \
  --instance-ids <instance-id> \
  --query "Reservations[*].Instances[*].[State.Name,InstanceType]" \
  --output table \
  --region us-east-1
```

## Important AWS CLI Commands Learned

- `aws ec2 wait instance-stopped` – Wait until the instance reaches the **stopped** state.
- `aws ec2 wait instance-running` – Wait until the instance reaches the **running** state.
- `aws ec2 modify-instance-attribute` – Modify EC2 attributes such as the instance type.
- `aws ec2 describe-instance-status` – Verify **System Status** and **Instance Status** before making changes.
