# Upgrade RDS MySQL Engine Version via AWS CLI

## Problem Statement

Upgrade an existing Amazon RDS MySQL instance:

- **RDS Instance:** `xfusion-rds`
- **Current Version:** MySQL `8.4.5`
- **Target Version:** MySQL `8.4.6`
- **Apply Changes:** Immediately
- **Final State:** `available`
- **Region:** `us-east-1`

## Steps

1. Verify the AWS CLI environment.
   ```bash
   aws sts get-caller-identity
   aws configure list
   aws configure get region
   ```

2. Verify the current RDS status and engine version.
   ```bash
   aws rds describe-db-instances \
     --db-instance-identifier xfusion-rds \
     --query "DBInstances[0].[DBInstanceIdentifier,DBInstanceStatus,Engine,EngineVersion]" \
     --output table \
     --region us-east-1
   ```

3. Verify that MySQL `8.4.6` exists.
   ```bash
   aws rds describe-db-engine-versions \
     --engine mysql \
     --engine-version 8.4.6 \
     --query "DBEngineVersions[*].[Engine,EngineVersion]" \
     --output table \
     --region us-east-1
   ```

4. Verify that `8.4.6` is a valid upgrade target from `8.4.5`.
   ```bash
   aws rds describe-db-engine-versions \
     --engine mysql \
     --engine-version 8.4.5 \
     --query "DBEngineVersions[0].ValidUpgradeTarget[*].[EngineVersion,IsMajorVersionUpgrade]" \
     --output table \
     --region us-east-1
   ```

5. Use AWS CLI help to discover the required modification options.
   ```bash
   aws rds modify-db-instance help | less
   ```

   Search inside `less`:
   ```text
   /engine-version
   /apply-immediately
   ```

6. Upgrade the RDS MySQL engine.
   ```bash
   aws rds modify-db-instance \
     --db-instance-identifier xfusion-rds \
     --engine-version 8.4.6 \
     --apply-immediately \
     --region us-east-1
   ```

7. Wait until the RDS instance returns to `available`.
   ```bash
   aws rds wait db-instance-available \
     --db-instance-identifier xfusion-rds \
     --region us-east-1
   ```

## Validation

Verify the final engine version and RDS state.

```bash
aws rds describe-db-instances \
  --db-instance-identifier xfusion-rds \
  --query "DBInstances[0].[DBInstanceIdentifier,DBInstanceStatus,Engine,EngineVersion]" \
  --output table \
  --region us-east-1
```

Expected result:

```text
xfusion-rds
available
mysql
8.4.6
```

## Important AWS CLI Commands Learned

- `describe-db-instances` – Check the current RDS state and engine version.
- `describe-db-engine-versions` – Inspect available versions and valid upgrade targets.
- `ValidUpgradeTarget` – Verify an upgrade path before modifying production infrastructure.
- `modify-db-instance` – Modify an existing RDS instance.
- `--engine-version` – Specify the target database engine version.
- `--apply-immediately` – Apply the modification immediately.
- `wait db-instance-available` – Wait for the upgrade to complete.
- `aws <service> <command> help` – Discover AWS CLI syntax instead of memorizing every option.

## Engineering Insight

Before upgrading a database, do not assume that a newer version is a valid upgrade path.

```text
Current State
     ↓
Verify Target Version Exists
     ↓
Verify ValidUpgradeTarget
     ↓
Discover Syntax with --help
     ↓
Apply Change
     ↓
Wait for Stable State
     ↓
Validate Original Requirement
```
