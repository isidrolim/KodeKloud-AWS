# Delete IAM Role via AWS CLI

## Problem Statement

Delete the existing IAM role:

- **IAM Role:** `iamrole_kareem`

> **Note:** Before deleting an IAM role, verify that no managed policies, inline policies, or instance profiles depend on it.

## Steps

1. Verify the AWS CLI environment.
   ```bash
   aws sts get-caller-identity
   aws configure list
   ```

2. Verify the IAM role exists.
   ```bash
   aws iam get-role \
     --role-name iamrole_kareem
   ```

3. Check for attached managed policies.
   ```bash
   aws iam list-attached-role-policies \
     --role-name iamrole_kareem \
     --output table
   ```

4. Check for inline policies.
   ```bash
   aws iam list-role-policies \
     --role-name iamrole_kareem \
     --output table
   ```

5. Check for associated instance profiles.
   ```bash
   aws iam list-instance-profiles-for-role \
     --role-name iamrole_kareem \
     --output table
   ```

6. If no dependencies exist, delete the IAM role.
   ```bash
   aws iam delete-role \
     --role-name iamrole_kareem
   ```

## Validation

Verify that the role no longer exists.

```bash
aws iam get-role \
  --role-name iamrole_kareem
```

Expected result:

```text
NoSuchEntity
```

## Important AWS CLI Commands Learned

- `get-role` – Verify and inspect an IAM role.
- `list-attached-role-policies` – Check managed policies attached to a role.
- `list-role-policies` – Check inline policies configured on a role.
- `list-instance-profiles-for-role` – Check whether the role belongs to an instance profile.
- `delete-role` – Delete an IAM role after its dependencies have been removed.
- AWS CLI parameters must be explicitly provided, e.g. `--role-name iamrole_kareem`, rather than passing the role name as a positional argument.
