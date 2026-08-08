# Create IAM Group via AWS CLI

## Problem Statement

Create an **IAM group** with the following requirement:

- **IAM Group Name:** `iamgroup_jim`

> **Note:** IAM is a global AWS service, so a Region does not need to be specified.

## Steps

1. Verify the AWS CLI environment.
   ```bash
   aws sts get-caller-identity
   aws configure list
   ```

2. List existing IAM groups.
   ```bash
   aws iam list-groups \
     --query "Groups[*].[GroupName,CreateDate]" \
     --output table
   ```

3. Check whether the group already exists.
   ```bash
   aws iam get-group \
     --group-name iamgroup_jim
   ```

4. Create the IAM group.
   ```bash
   aws iam create-group \
     --group-name iamgroup_jim
   ```

## Validation

Verify the IAM group exists.

```bash
aws iam get-group \
  --group-name iamgroup_jim
```

Or list all IAM groups:

```bash
aws iam list-groups \
  --query "Groups[*].[GroupName,CreateDate]" \
  --output table
```

Expected result:

```text
GroupName: iamgroup_jim
```

## Important AWS CLI Commands Learned

- `aws iam list-groups` – List IAM groups in the account.
- `aws iam get-group` – Retrieve a specific IAM group and its members.
- `aws iam create-group` – Create a new IAM group.
- IAM groups are used to manage permissions for multiple IAM users.
- IAM is a **global AWS service**, so `--region` is normally unnecessary.
