# Attach EBS Volume to EC2 Instance via AWS CLI

## Problem Statement

Attach an existing **Amazon EBS Volume** to an existing EC2 instance with the following requirements:

- **EC2 Instance:** `datacenter-ec2`
- **EBS Volume:** `datacenter-volume`
- **Device Name:** `/dev/sdb`
- **Region:** `us-east-1`

> **Note:** Ensure the volume is in the **available** state and in the **same Availability Zone** as the EC2 instance before attaching it.

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
     --filters "Name=tag:Name,Values=datacenter-ec2" \
     --query "Reservations[*].Instances[*].[InstanceId,State.Name,Placement.AvailabilityZone]" \
     --output table \
     --region us-east-1
   ```

5. Locate the EBS volume.
   ```bash
   aws ec2 describe-volumes \
     --filters "Name=tag:Name,Values=datacenter-volume" \
     --query "Volumes[*].[VolumeId,State,AvailabilityZone,Size,VolumeType]" \
     --output table \
     --region us-east-1
   ```

6. Verify the volume is not attached.
   ```bash
   aws ec2 describe-volumes \
     --filters "Name=tag:Name,Values=datacenter-volume" \
     --query "Volumes[*].Attachments" \
     --output json \
     --region us-east-1
   ```

7. Attach the volume to the EC2 instance.
   ```bash
   aws ec2 attach-volume \
     --volume-id <volume-id> \
     --instance-id <instance-id> \
     --device /dev/sdb \
     --region us-east-1
   ```

## Validation

Verify the volume attachment.

```bash
aws ec2 describe-volumes \
  --volume-ids <volume-id> \
  --query "Volumes[0].[State,Attachments[0].State,Attachments[0].InstanceId,Attachments[0].Device]" \
  --output table \
  --region us-east-1
```

Expected output:

```text
State:            in-use
Attachment.State: attached
InstanceId:       <instance-id>
Device:           /dev/sdb
```

## Important AWS CLI Commands Learned

- `describe-volumes` – Retrieve EBS volume information.
- `attach-volume` – Attach an EBS volume to an EC2 instance.
- `Attachments` – Shows the attachment details of an EBS volume.
- `--device` – Specifies the Linux device name presented to the EC2 instance.
- **Important:** An EBS volume can only be attached to an EC2 instance in the **same Availability Zone**.
