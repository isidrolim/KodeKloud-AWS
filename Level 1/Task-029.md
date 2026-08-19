# Create RDS DB Snapshot via AWS CLI

## Problem Statement

Create a manual snapshot of an existing Amazon RDS instance:

- **RDS Instance:** `xfusion-rds`
- **Snapshot Name:** `xfusion-rds-snapshot`
- **Region:** `us-east-1`
- **Required Snapshot State:** `available`

## Steps

1. Verify the AWS CLI environment.
   ```bash
   aws sts get-caller-identity
   aws configure list
   aws configure get region
   ```

2. Check the RDS instance and its current status.
   ```bash
   aws rds describe-db-instances \
     --db-instance-identifier xfusion-rds \
     --query "DBInstances[0].[DBInstanceIdentifier,DBInstanceStatus,Engine,EngineVersion]" \
     --output table \
     --region us-east-1
   ```

3. If the instance is still `creating`, wait until it becomes `available`.
   ```bash
   aws rds wait db-instance-available \
     --db-instance-identifier xfusion-rds \
     --region us-east-1
   ```

4. Check whether the snapshot already exists.
   ```bash
   aws rds describe-db-snapshots \
     --db-snapshot-identifier xfusion-rds-snapshot \
     --query "DBSnapshots[*].[DBSnapshotIdentifier,DBInstanceIdentifier,Status,SnapshotType]" \
     --output table \
     --region us-east-1
   ```

5. Create the manual RDS snapshot.
   ```bash
   aws rds create-db-snapshot \
     --db-instance-identifier xfusion-rds \
     --db-snapshot-identifier xfusion-rds-snapshot \
     --region us-east-1
   ```

6. Wait until the snapshot becomes available.
   ```bash
   aws rds wait db-snapshot-available \
     --db-snapshot-identifier xfusion-rds-snapshot \
     --region us-east-1
   ```

## Validation

Verify the final snapshot status.

```bash
aws rds describe-db-snapshots \
  --db-snapshot-identifier xfusion-rds-snapshot \
  --query "DBSnapshots[0].[DBSnapshotIdentifier,DBInstanceIdentifier,Status,Engine,SnapshotType]" \
  --output table \
  --region us-east-1
```

Expected result:

```text
Snapshot: xfusion-rds-snapshot
RDS:      xfusion-rds
Status:   available
Engine:   mysql
Type:     manual
```

## Important AWS CLI Commands Learned

- `describe-db-instances` – Check the RDS instance and its current state.
- `wait db-instance-available` – Wait until an RDS instance is ready.
- `describe-db-snapshots` – Find and inspect RDS snapshots.
- `create-db-snapshot` – Create a manual snapshot of an RDS instance.
- `wait db-snapshot-available` – Wait until snapshot creation completes.
- Always verify the source RDS instance is `available` before creating the snapshot.
- RDS snapshot lifecycle: `creating` → `available`.
