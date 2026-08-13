# Delete IAM Group via AWS CLI

## Problem Statement

Delete the existing IAM group:

- **IAM Group:** `iamgroup_siva`

> **Note:** Before deleting an IAM group, verify that it has no users or policies that must be removed first.

## Steps

1. Verify the AWS CLI environment.
   ```bash
   aws sts get-caller-identity
   aws configure list
   ```

2. Verify the IAM group exists and check its members.
   ```bash
   aws iam get-group \
     --group-name iamgroup_siva
   ```

3. Check for attached managed policies.
   ```bash
   aws iam list-attached-group-policies \
     --group-name iamgroup_siva \
     --output table
   ```

4. Check for inline policies.
   ```bash
   aws iam list-group-policies \
     --group-name iamgroup_siva \
     --output table
   ```

5. If dependencies exist, remove them before deleting the group.

   Remove a user:
   ```bash
   aws iam remove-user-from-group \
     --group-name iamgroup_siva \
     --user-name <user-name>
   ```

   Detach a managed policy:
   ```bash
   aws iam detach-group-policy \
     --group-name iamgroup_siva \
     --policy-arn <policy-arn>
   ```

   Delete an inline policy:
   ```bash
   aws iam delete-group-policy \
     --group-name iamgroup_siva \
     --policy-name <policy-name>
   ```

6. Delete the IAM group.
   ```bash
   aws iam delete-group \
     --group-name iamgroup_siva
   ```

## Validation

Verify the group no longer exists.

```bash
aws iam get-group \
  --group-name iamgroup_siva
```

Expected result:

```text
NoSuchEntity
```

## Important AWS CLI Commands Learned

- `get-group` – Verify a group and view its members.
- `list-attached-group-policies` – Check managed policies attached to a group.
- `list-group-policies` – Check inline policies assigned to a group.
- `remove-user-from-group` – Remove a user before deleting a group.
- `detach-group-policy` – Detach a managed policy.
- `delete-group-policy` – Delete an inline policy; requires `--policy-name`.
- `delete-group` – Delete the IAM group after its dependencies are removed.
