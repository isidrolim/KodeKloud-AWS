# Create Publicly Accessible RDS MySQL Instance via AWS CLI

## Problem Statement

Create an Amazon RDS instance with the following requirements:

- **DB Instance Identifier:** `devops-rds`
- **Engine:** MySQL `8.4.5`
- **Instance Class:** `db.t3.micro`
- **Master Username:** `devops_admin`
- **Storage:** `5 GiB`
- **Storage Type:** `gp2`
- **Publicly Accessible:** Yes
- **Final State:** `available`
- **Region:** `us-east-1`

## Steps

1. Verify the AWS CLI environment.
   ```bash
   aws sts get-caller-identity
   aws configure list
   aws configure get region
   ```

2. Check whether the RDS instance already exists.
   ```bash
   aws rds describe-db-instances \
     --db-instance-identifier devops-rds \
     --region us-east-1
   ```

3. Verify MySQL `8.4.5` is available.
   ```bash
   aws rds describe-db-engine-versions \
     --engine mysql \
     --engine-version 8.4.5 \
     --query "DBEngineVersions[*].[Engine,EngineVersion]" \
     --output table \
     --region us-east-1
   ```

4. Verify `db.t3.micro` and `gp2` are supported.
   ```bash
   aws rds describe-orderable-db-instance-options \
     --engine mysql \
     --engine-version 8.4.5 \
     --db-instance-class db.t3.micro \
     --query "OrderableDBInstanceOptions[*].[DBInstanceClass,Engine,EngineVersion,StorageType]" \
     --output table \
     --region us-east-1
   ```

5. Set a database password.
   ```bash
   DB_PASSWORD='YourStrongPassword123!'
   ```

6. Create the RDS instance.
   ```bash
   aws rds create-db-instance \
     --db-instance-identifier devops-rds \
     --db-instance-class db.t3.micro \
     --engine mysql \
     --engine-version 8.4.5 \
     --master-username devops_admin \
     --master-user-password "$DB_PASSWORD" \
     --allocated-storage 5 \
     --storage-type gp2 \
     --publicly-accessible \
     --region us-east-1
   ```

7. Wait until the RDS instance becomes available.
   ```bash
   aws rds wait db-instance-available \
     --db-instance-identifier devops-rds \
     --region us-east-1
   ```

## Validation

Verify all required RDS settings.

```bash
aws rds describe-db-instances \
  --db-instance-identifier devops-rds \
  --query "DBInstances[0].[DBInstanceIdentifier,DBInstanceStatus,Engine,EngineVersion,DBInstanceClass,AllocatedStorage,StorageType,MasterUsername,PubliclyAccessible]" \
  --output table \
  --region us-east-1
```

Expected result:

```text
DB Identifier:       devops-rds
Status:              available
Engine:              mysql
Engine Version:      8.4.5
Instance Class:      db.t3.micro
Storage:             5 GiB
Storage Type:        gp2
Master Username:     devops_admin
Publicly Accessible: True
```

## Important AWS CLI Commands Learned

- `describe-db-instances` – Find and inspect RDS instances.
- `describe-db-engine-versions` – Verify available database engine versions.
- `describe-orderable-db-instance-options` – Verify supported engine, instance class, and storage combinations.
- `create-db-instance` – Provision a new RDS database instance.
- `--publicly-accessible` – Configure the RDS instance with a public endpoint.
- `wait db-instance-available` – Wait until RDS provisioning is complete.
- RDS provisioning is asynchronous: `creating` → `backing-up` → `available`.
- Always verify engine, instance class, storage, accessibility, and final state after provisioning.
