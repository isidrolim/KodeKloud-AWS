# Create EBS Volume Snapshot via AWS CLI

## Problem Statement

Create a snapshot from an existing EBS volume with the following requirements:

- **Source Volume:** `datacenter-vol`
- **Snapshot Name:** `datacenter-vol-ss`
- **Description:** `datacenter Snapshot`
- **Required State:** `completed`
- **Region:** `us-east-1`

## Steps

1. Verify the AWS CLI environment.
   ```bash
   aws sts get-caller-identity
   aws configure list
   aws configure get region
   ```

2. Locate the source EBS volume.
   ```bash
   aws ec2 describe-volumes \
     --filters "Name=tag:Name,Values=datacenter-vol" \
     --query "Volumes[*].[VolumeId,State,AvailabilityZone,Size,VolumeType]" \
     --output table \
     --region us-east-1
   ```

3. Check whether the snapshot already exists.
   ```bash
   aws ec2 describe-snapshots \
     --owner-ids self \
     --filters "Name=tag:Name,Values=datacenter-vol-ss" \
     --query "Snapshots[*].[SnapshotId,VolumeId,State,Description]" \
     --output table \
     --region us-east-1
   ```

4. Create the snapshot and apply the required Name tag.
   ```bash
   aws ec2 create-snapshot \
     --volume-id <volume-id> \
     --description "datacenter Snapshot" \
     --tag-specifications 'ResourceType=snapshot,Tags=[{Key=Name,Value=datacenter-vol-ss}]' \
     --region us-east-1
   ```

5. Wait until snapshot creation is completed.
   ```bash
   aws ec2 wait snapshot-completed \
     --snapshot-ids <snapshot-id> \
     --region us-east-1
   ```

## Validation

Verify the snapshot status, source volume, description, and Name tag.

```bash
aws ec2 describe-snapshots \
  --snapshot-ids <snapshot-id> \
  --query "Snapshots[0].[SnapshotId,VolumeId,State,Description,Tags[?Key=='Name']|[0].Value]" \
  --output table \
  --region us-east-1
```

Expected result:

```text
State:       completed
Name:        datacenter-vol-ss
Description: datacenter Snapshot
```

## Important AWS CLI Commands Learned

- `describe-volumes` – Locate and inspect EBS volumes.
- `describe-snapshots` – Inspect existing EBS snapshots.
- `create-snapshot` – Create a point-in-time snapshot of an EBS volume.
- `--owner-ids self` – Limit snapshot results to the current AWS account.
- `wait snapshot-completed` – Wait until AWS finishes creating the snapshot.
- Snapshot lifecycle: `pending` → `completed`.
