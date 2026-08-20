# Delete RDS Instance via AWS CLI

## Problem Statement

Delete an existing Amazon RDS instance that is no longer required:

- **RDS Instance:** `datacenter-rds`
- **Region:** `us-east-1`
- **Required Final State:** Deleted / no longer exists

## Steps

1. Verify the AWS CLI environment.
   ```bash
   aws sts get-caller-identity
   aws configure list
   aws configure get region
   ```

2. Verify the target RDS instance exists and inspect its state.
   ```bash
   aws rds describe-db-instances \
     --db-instance-identifier datacenter-rds \
     --query "DBInstances[0].[DBInstanceIdentifier,DBInstanceStatus,DeletionProtection]" \
     --output table \
     --region us-east-1
   ```

3. Use AWS CLI help to understand the deletion options before performing the destructive operation.
   ```bash
   aws rds delete-db-instance help | less
   ```

   Useful options to look for:
   ```text
   --db-instance-identifier
   --skip-final-snapshot
   --final-db-snapshot-identifier
   ```

4. Delete the RDS instance.

   If the task does not require a final snapshot:
   ```bash
   aws rds delete-db-instance \
     --db-instance-identifier datacenter-rds \
     --skip-final-snapshot \
     --region us-east-1
   ```

5. Observe the deletion state.
   ```bash
   aws rds describe-db-instances \
     --db-instance-identifier datacenter-rds \
     --query "DBInstances[0].[DBInstanceIdentifier,DBInstanceStatus]" \
     --output table \
     --region us-east-1
   ```

   During deletion, the expected transitional state is:
   ```text
   datacenter-rds
   deleting
   ```

6. Instead of repeatedly checking manually, AWS CLI also provides a waiter:
   ```bash
   aws rds wait db-instance-deleted \
     --db-instance-identifier datacenter-rds \
     --region us-east-1
   ```

## Validation

Verify that the RDS instance no longer exists.

```bash
aws rds describe-db-instances \
  --db-instance-identifier datacenter-rds \
  --region us-east-1
```

Expected result:

```text
DBInstanceNotFound
```

This error is the successful validation condition because the original requirement was to completely remove the RDS instance.

## Important AWS CLI Commands Learned

- `describe-db-instances` – Verify an RDS instance and inspect its current state.
- `delete-db-instance` – Delete an existing RDS instance.
- `--skip-final-snapshot` – Delete without creating a final DB snapshot.
- `--final-db-snapshot-identifier` – Specify a final snapshot when one must be retained.
- `wait db-instance-deleted` – Wait until AWS confirms the database has been completely removed.
- `DBInstanceNotFound` after deletion is expected evidence that the resource no longer exists.

## Engineering Insight

For destructive operations, always follow:

```text
Identify exact resource
        ↓
Inspect current state
        ↓
Check protection/dependencies
        ↓
Read command help
        ↓
Understand data-loss options
        ↓
Execute deletion
        ↓
Observe transitional state
        ↓
Validate resource no longer exists
```

Do not treat `DBInstanceNotFound` as a failure when your success criterion is deletion. In this case, **not found is proof that the requested cleanup succeeded**.

Also, using:

```bash
aws rds delete-db-instance help
```

before executing the command is more valuable than memorizing every flag. It lets you discover the correct syntax and understand the operational consequences before performing a destructive action.
