# Create Read-Only EC2 IAM Policy via AWS CLI

## Problem Statement

Create a custom **IAM policy** with the following requirements:

- **Policy Name:** `iampolicy_rose`
- Allow read-only access to view:
  - EC2 Instances
  - AMIs
  - EBS Snapshots

> **Note:** IAM policies are global resources, so `--region` is not required.

## Steps

1. Verify the AWS CLI environment.
   ```bash
   aws sts get-caller-identity
   aws configure list
   ```

2. Check whether the policy already exists.
   ```bash
   aws iam list-policies \
     --scope Local \
     --query "Policies[?PolicyName=='iampolicy_rose'].[PolicyName,Arn]" \
     --output table
   ```

3. Create the IAM policy document.
   ```bash
   cat > iampolicy_rose.json <<'EOF'
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": [
           "ec2:DescribeInstances",
           "ec2:DescribeImages",
           "ec2:DescribeSnapshots"
         ],
         "Resource": "*"
       }
     ]
   }
   EOF
   ```

4. Verify the policy document.
   ```bash
   cat iampolicy_rose.json
   ```

5. Create the IAM policy.
   ```bash
   aws iam create-policy \
     --policy-name iampolicy_rose \
     --policy-document file://iampolicy_rose.json
   ```

## Validation

Verify the custom policy exists.

```bash
aws iam list-policies \
  --scope Local \
  --query "Policies[?PolicyName=='iampolicy_rose'].[PolicyName,Arn,DefaultVersionId]" \
  --output table
```

## Important AWS CLI Commands Learned

- `list-policies` – List IAM managed policies.
- `--scope Local` – Show customer-managed policies from the current AWS account.
- `create-policy` – Create a customer-managed IAM policy.
- `--policy-document file://...` – Load an IAM policy from a local JSON file.
- `ec2:DescribeInstances` – Read EC2 instance information.
- `ec2:DescribeImages` – Read AMI information.
- `ec2:DescribeSnapshots` – Read EBS snapshot information.
- `"Resource": "*"` – Applies these read-only actions across applicable EC2 resources.
