# Enable EC2 Termination Protection via AWS CLI

## Problem Statement

Enable **Termination Protection** for the EC2 instance with the following requirements:

- **Instance Name:** `xfusion-ec2`
- **Region:** `us-east-1`

> **Note:** Termination Protection prevents accidental deletion of an EC2 instance through the AWS Console or API.

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

5. Verify the current Termination Protection status.
   ```bash
   aws ec2 describe-instance-attribute \
     --instance-id <instance-id> \
     --attribute disableApiTermination \
     --region us-east-1
   ```

6. Enable Termination Protection.
   ```bash
   aws ec2 modify-instance-attribute \
     --instance-id <instance-id> \
     --disable-api-termination "{\"Value\":true}" \
     --region us-east-1
   ```

## Validation

Verify that Termination Protection is enabled.

```bash
aws ec2 describe-instance-attribute \
  --instance-id <instance-id> \
  --attribute disableApiTermination \
  --region us-east-1
```

Expected output:

```json
{
    "DisableApiTermination": {
        "Value": true
    }
}
```

## Important AWS CLI Commands Learned

- `describe-instance-attribute` – Retrieve a specific EC2 instance attribute.
- `modify-instance-attribute` – Modify EC2 instance attributes.
- `disableApiTermination` – Controls **Termination Protection**.
- `--disable-api-termination` – Enables or disables API termination protection.
- `--attribute` – Reads a specific EC2 attribute instead of the entire instance configuration.
