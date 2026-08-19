# Enable RDS Deletion Protection via AWS CLI

## Problem Statement

Enable deletion protection for an existing Amazon RDS instance:

- **RDS Instance:** `datacenter-rds`
- **Deletion Protection:** Enabled
- **Apply Changes:** Immediately
- **Region:** `us-east-1`

## Steps

1. Verify the AWS CLI environment.
   ```bash
   aws sts get-caller-identity
   aws configure list
   aws configure get region
   ```

2. Check the current RDS status and deletion protection setting.
   ```bash
   aws rds describe-db-instances \
     --db-instance-identifier datacenter-rds \
     --query "DBInstances[0].[DBInstanceIdentifier,DBInstanceStatus,DeletionProtection]" \
     --output table \
     --region us-east-1
   ```

3. Enable deletion protection and apply the change immediately.
   ```bash
   aws rds modify-db-instance \
     --db-instance-identifier datacenter-rds \
     --deletion-protection \
     --apply-immediately \
     --region us-east-1
   ```

4. Wait until the RDS instance returns to the `available` state.
   ```bash
   aws rds wait db-instance-available \
     --db-instance-identifier datacenter-rds \
     --region us-east-1
   ```

## Validation

Verify that deletion protection is enabled.

```bash
aws rds describe-db-instances \
  --db-instance-identifier datacenter-rds \
  --query "DBInstances[0].[DBInstanceIdentifier,DBInstanceStatus,DeletionProtection]" \
  --output table \
  --region us-east-1
```

Expected result:

```text
datacenter-rds
available
True
```

## Important AWS CLI Commands Learned

- `describe-db-instances` – Inspect an RDS instance and its configuration.
- `modify-db-instance` – Modify an existing RDS instance.
- `--deletion-protection` – Enable protection against accidental RDS deletion.
- `--apply-immediately` – Apply the requested modification immediately.
- `wait db-instance-available` – Wait until the RDS instance returns to a stable `available` state.
- Always verify destructive-operation protection on production databases.
