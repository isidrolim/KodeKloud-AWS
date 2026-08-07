# Create IAM User via AWS CLI

## Problem Statement

Create an **IAM user** with the following requirement:

- **IAM User Name:** `iamuser_javed`

> **Note:** IAM is a global AWS service, so a Region does not need to be specified when managing IAM users.

## Steps

1. Verify the AWS CLI is authenticated.
   ```bash
   aws sts get-caller-identity
   ```

2. Verify the AWS CLI configuration.
   ```bash
   aws configure list
   ```

3. List existing IAM users.
   ```bash
   aws iam list-users \
     --query "Users[*].[UserName,CreateDate]" \
     --output table
   ```

4. Check specifically whether the user already exists.
   ```bash
   aws iam get-user \
     --user-name iamuser_javed
   ```

   If the user does not exist, AWS returns `NoSuchEntity`.

5. Create the IAM user.
   ```bash
   aws iam create-user \
     --user-name iamuser_javed
   ```

## Validation

Verify the new IAM user.

```bash
aws iam get-user \
  --user-name iamuser_javed
```

Optionally list all IAM users again.

```bash
aws iam list-users \
  --query "Users[*].[UserName,CreateDate]" \
  --output table
```

## Important AWS CLI Commands Learned

- `aws iam list-users` – List IAM users in the AWS account.
- `aws iam get-user` – Retrieve a specific IAM user.
- `aws iam create-user` – Create a new IAM user.
- IAM is a **global service**, unlike regional resources such as EC2, EBS, and VPC.
