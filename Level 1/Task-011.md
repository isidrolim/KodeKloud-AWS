# Attach Elastic Network Interface (ENI) via AWS CLI

## Problem Statement

Attach an existing **Elastic Network Interface (ENI)** to an existing EC2 instance with the following requirements:

- **EC2 Instance:** `xfusion-ec2`
- **Elastic Network Interface:** `xfusion-eni`
- **Region:** `us-east-1`

> **Note:** Ensure the ENI status is **attached** before submitting the task.

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
     --query "Reservations[*].Instances[*].[InstanceId,State.Name,Placement.AvailabilityZone]" \
     --output table \
     --region us-east-1
   ```

5. Locate the ENI and verify it is available.
   ```bash
   aws ec2 describe-network-interfaces \
     --filters "Name=tag:Name,Values=xfusion-eni" \
     --query "NetworkInterfaces[*].[NetworkInterfaceId,Status,AvailabilityZone,Attachment.InstanceId,PrivateIpAddress]" \
     --output table \
     --region us-east-1
   ```

6. Attach the ENI as the secondary network interface (`DeviceIndex=1`).
   ```bash
   aws ec2 attach-network-interface \
     --network-interface-id <eni-id> \
     --instance-id <instance-id> \
     --device-index 1 \
     --region us-east-1
   ```

## Validation

Verify the ENI is attached to the EC2 instance.

```bash
aws ec2 describe-network-interfaces \
  --network-interface-ids <eni-id> \
  --query "NetworkInterfaces[0].[Status,Attachment.Status,Attachment.InstanceId,Attachment.DeviceIndex]" \
  --output table \
  --region us-east-1
```

Expected output:

```text
Status:             in-use
Attachment.Status:  attached
InstanceId:         <instance-id>
DeviceIndex:        1
```

## Important AWS CLI Commands Learned

- `describe-network-interfaces` – List ENIs and their current status.
- `attach-network-interface` – Attach an existing ENI to an EC2 instance.
- `--device-index` – Specifies which network interface slot to use (`0` = primary, `1` = secondary).
- `Attachment.Status` – Shows whether the ENI is successfully attached.
- `NetworkInterfaceId (eni-xxxx)` – Unique identifier of an Elastic Network Interface.
