# Create GP3 EBS Volume via AWS CLI

## Problem Statement

Create an **Amazon EBS Volume** with the following requirements:

- **Volume Name:** `datacenter-volume`
- **Volume Type:** `gp3`
- **Volume Size:** `2 GiB`
- **Region:** `us-east-1`
- **Availability Zone:** `us-east-1a`

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

4. List the available Availability Zones.
   ```bash
   aws ec2 describe-availability-zones \
     --query "AvailabilityZones[*].ZoneName" \
     --output table \
     --region us-east-1
   ```

5. Check whether the volume already exists.
   ```bash
   aws ec2 describe-volumes \
     --filters "Name=tag:Name,Values=datacenter-volume" \
     --query "Volumes[*].[VolumeId,State]" \
     --output table \
     --region us-east-1
   ```

6. Create the GP3 volume.
   ```bash
   aws ec2 create-volume \
     --availability-zone us-east-1a \
     --size 2 \
     --volume-type gp3 \
     --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=datacenter-volume}]' \
     --region us-east-1
   ```

## Validation

Verify the volume properties.

```bash
aws ec2 describe-volumes \
  --filters "Name=tag:Name,Values=datacenter-volume" \
  --query "Volumes[*].[VolumeId,AvailabilityZone,VolumeType,Size,State]" \
  --output table \
  --region us-east-1
```
