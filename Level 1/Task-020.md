# Create IAM Role for EC2 and Attach Policy via AWS CLI

## Problem Statement

Create an **IAM role** with the following requirements:

- **Role Name:** `iamrole_siva`
- **Trusted Entity:** AWS Service
- **Use Case:** EC2
- **Policy:** `iampolicy_siva`

> **Note:** IAM is a global AWS service, so `--region` is not required.

## Steps

1. Verify the AWS CLI environment.
   ```bash
   aws sts get-caller-identity
   aws configure list
   ```

2. Verify the IAM policy exists and retrieve its ARN.
   ```bash
   aws iam list-policies \
     --scope Local \
     --query "Policies[?PolicyName=='iampolicy_siva'].[PolicyName,Arn]" \
     --output table
   ```

3. Check whether the IAM role already exists.
   ```bash
   aws iam get-role \
     --role-name iamrole_siva
   ```

4. Create the EC2 trust policy.
   ```bash
   cat > ec2-trust-policy.json <<'EOF'
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "Service": "ec2.amazonaws.com"
         },
         "Action": "sts:AssumeRole"
       }
     ]
   }
   EOF
   ```

5. Create the IAM role using the trust policy.
   ```bash
   aws iam create-role \
     --role-name iamrole_siva \
     --assume-role-policy-document file://ec2-trust-policy.json
   ```

6. Attach the existing policy to the role.
   ```bash
   aws iam attach-role-policy \
     --role-name iamrole_siva \
     --policy-arn <policy-arn>
   ```

## Validation

Verify the role and its EC2 trust relationship.

```bash
aws iam get-role \
  --role-name iamrole_siva
```

Verify the permission policy is attached.

```bash
aws iam list-attached-role-policies \
  --role-name iamrole_siva \
  --output table
```

Expected:

```text
Role:            iamrole_siva
Trusted Service: ec2.amazonaws.com
Policy:          iampolicy_siva
```

## Important AWS CLI Commands Learned

- `get-role` – Verify and inspect an IAM role.
- `create-role` – Create an IAM role with a trust policy.
- `attach-role-policy` – Attach a managed permission policy to a role.
- `list-attached-role-policies` – Verify policies attached to a role.
- `sts:AssumeRole` – Allows the trusted principal to assume the role.

### IAM Role Concept

```text
Trust Policy
    ↓
WHO can assume the role?
    ↓
EC2 (ec2.amazonaws.com)

Permission Policy
    ↓
WHAT can the role do?
    ↓
iampolicy_siva
```

### IAM Policy Attachment Pattern

```text
User  → attach-user-policy
Group → attach-group-policy
Role  → attach-role-policy
```
