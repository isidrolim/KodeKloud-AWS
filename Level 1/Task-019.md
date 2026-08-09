# Attach IAM Policy to IAM User via AWS CLI

## Problem Statement

Attach an existing IAM policy to an existing IAM user:

- **IAM User:** `iamuser_james`
- **IAM Policy:** `iampolicy_james`

> **Note:** IAM is a global AWS service, so `--region` is not required.

## Steps

1. Verify the AWS CLI environment.
   ```bash
   aws sts get-caller-identity
   aws configure list
   ```

2. Verify the IAM user exists.
   ```bash
   aws iam get-user \
     --user-name iamuser_james
   ```

3. Find the policy and retrieve its ARN.
   ```bash
   aws iam list-policies \
     --scope Local \
     --query "Policies[?PolicyName=='iampolicy_james'].[PolicyName,Arn]" \
     --output table
   ```

4. Check whether the policy is already attached.
   ```bash
   aws iam list-attached-user-policies \
     --user-name iamuser_james \
     --output table
   ```

5. Attach the policy to the IAM user.
   ```bash
   aws iam attach-user-policy \
     --user-name iamuser_james \
     --policy-arn <policy-arn>
   ```

## Validation

Verify the policy is attached to the user.

```bash
aws iam list-attached-user-policies \
  --user-name iamuser_james \
  --output table
```

Expected result:

```text
User:   iamuser_james
Policy: iampolicy_james
```

## Important AWS CLI Commands Learned

- `get-user` – Verify a specific IAM user exists.
- `list-policies` – Find an IAM policy and retrieve its ARN.
- `list-attached-user-policies` – View managed policies attached to a user.
- `attach-user-policy` – Attach a managed IAM policy to an IAM user.
- Managed policies are attached using their **Policy ARN**, not only their name.

### IAM Attachment Pattern

```text
User  → attach-user-policy
Group → attach-group-policy
Role  → attach-role-policy
```
