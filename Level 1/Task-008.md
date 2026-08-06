# Enable EC2 Stop Protection via AWS CLI

## Problem Statement

Enable **Stop Protection** for the EC2 instance with the following requirements:

- **Instance Name:** `xfusion-ec2`
- **Region:** `us-east-1`

> **Note:** Stop Protection prevents accidental API or Console stop operations.

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
     --filters "Name=tag:Name,Values=xfusion-ec2" \
     --query "Reservations[*].Instances[*].[InstanceId,State.Name]" \
     --output table \
     --region us-east-1
   ```

5. Verify the current Stop Protection status.
   ```bash
   aws ec2 describe-instance-attribute \
     --instance-id <instance-id> \
     --attribute disableApiStop \
     --region us-east-1
   ```

6. Enable Stop Protection.
   ```bash
   aws ec2 modify-instance-attribute \
     --instance-id <instance-id> \
     --disable-api-stop "{\"Value\":true}" \
     --region us-east-1
   ```

## Validation

Verify that Stop Protection is enabled.

```bash
aws ec2 describe-instance-attribute \
  --instance-id <instance-id> \
  --attribute disableApiStop \
  --region us-east-1
```

Expected output:

```json
{
    "DisableApiStop": {
        "Value": true
    }
}
```

## Important AWS CLI Commands Learned

- `describe-instance-attribute` – View a specific EC2 instance attribute.
- `modify-instance-attribute` – Modify EC2 instance attributes.
- `disableApiStop` – Controls **Stop Protection** (prevents accidental stop operations).
- `--attribute` – Retrieves a specific attribute instead of the entire resource.
