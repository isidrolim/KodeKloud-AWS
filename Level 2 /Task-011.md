# AWS Level 2 – Task 011: Snapshot and Restoration of an RDS Instance

## Scenario
The Nautilus Development Team needs to create a manual snapshot of an existing RDS instance and restore it to a new RDS instance to verify the database backup and recovery process.

## Requirements

- **Region:** `us-east-1`
- **Source RDS Instance:** `devops-rds`
- **Snapshot Name:** `devops-snapshot`
- **Restored RDS Instance:** `devops-snapshot-restore`
- **Restored Instance Class:** `db.t3.micro`
- Source RDS must be `Available` before taking the snapshot
- Restored RDS instance must reach `Available` state

## Steps

### 1. Verify the Source RDS Instance

Go to:

`AWS Console → RDS → Databases → devops-rds`

Confirm that the instance status is:

`Available`

Do not create the snapshot while the database is still being created or modified.

---

### 2. Create the RDS Snapshot

Select:

`devops-rds → Actions → Take snapshot`

Configure:

- **Snapshot name:** `devops-snapshot`

Click:

`Take snapshot`

---

### 3. Wait for the Snapshot

Go to:

`RDS → Snapshots → Manual`

Locate:

`devops-snapshot`

Wait until its status becomes:

`Available`

The snapshot must be fully available before starting the restore.

---

### 4. Restore the Snapshot

Select:

`devops-snapshot → Actions → Restore snapshot`

Configure the new database:

- **DB instance identifier:** `devops-snapshot-restore`
- **DB instance class:** `db.t3.micro`

Keep the remaining configuration at the appropriate/default values unless the lab requires otherwise.

Review the settings and click:

`Restore DB instance`

---

### 5. Wait for the Restored RDS Instance

Go to:

`RDS → Databases`

Locate:

`devops-snapshot-restore`

The instance will initially show:

`Creating`

Wait until it becomes:

`Available`

---

## Validation

Verify the original database:

`devops-rds → Available`

Verify the manual snapshot:

`devops-snapshot → Available`

Verify the restored database:

`devops-snapshot-restore → Available`

Confirm the restored instance class is:

`db.t3.micro`

The backup and recovery workflow is:

`devops-rds → devops-snapshot → Restore → devops-snapshot-restore`

## Result

A manual snapshot named `devops-snapshot` was successfully created from the existing `devops-rds` RDS instance.

The snapshot was then restored as a new RDS instance named `devops-snapshot-restore` using the `db.t3.micro` instance class, and the restored database reached the `Available` state.

**Task Status:** ✅ Completed
