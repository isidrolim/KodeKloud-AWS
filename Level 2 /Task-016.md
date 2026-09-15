# AWS Level 2 – Task 016: Create a Lambda Function Using AWS CLI

## Scenario
The Nautilus DevOps team needs to deploy a Python AWS Lambda function using the AWS CLI from the `aws-client` host.

## Requirements

- **Function:** `devops-lambda-cli`
- **Python file:** `lambda_function.py`
- **ZIP file:** `function.zip`
- **Runtime:** Python
- **IAM Role:** `lambda_execution_role`
- **Status Code:** `200`
- **Body:** `Welcome to KKE AWS Labs!`

## Steps

### 1. Create the Python Script

```bash
cat > lambda_function.py <<'EOF'
def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "body": "Welcome to KKE AWS Labs!"
    }
EOF
```

### 2. Package the Lambda Code

```bash
zip function.zip lambda_function.py
```

Verify:

```bash
unzip -l function.zip
```

### 3. Get the IAM Role ARN

```bash
ROLE_ARN=$(aws iam get-role \
  --role-name lambda_execution_role \
  --query 'Role.Arn' \
  --output text)

echo "$ROLE_ARN"
```

### 4. Create the Lambda Function

```bash
aws lambda create-function \
  --function-name devops-lambda-cli \
  --runtime python3.13 \
  --role "$ROLE_ARN" \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip
```

### 5. Verify the Lambda Configuration

```bash
aws lambda get-function-configuration \
  --function-name devops-lambda-cli \
  --query '[FunctionName,Runtime,Handler,Role,State]' \
  --output table
```

Confirmed:

```text
Function: devops-lambda-cli
Runtime:  python3.13
Handler:  lambda_function.lambda_handler
Role:     lambda_execution_role
State:    Active
```

### 6. Invoke the Function

The lab host uses AWS CLI v1, so invoke the function with:

```bash
aws lambda invoke \
  --function-name devops-lambda-cli \
  --payload '{}' \
  response.json
```

Check the result:

```bash
cat response.json
```

Expected response:

```json
{
    "statusCode": 200,
    "body": "Welcome to KKE AWS Labs!"
}
```

## Validation

- Lambda function `devops-lambda-cli` exists
- Runtime is Python
- Handler is `lambda_function.lambda_handler`
- IAM role is `lambda_execution_role`
- Function state is `Active`
- Invocation succeeds
- Response returns status code `200`
- Response body is `Welcome to KKE AWS Labs!`

## Lessons Learned

- Lambda deployment packages can be created locally and deployed directly using AWS CLI.
- The Lambda handler must match the Python file and function name.
- AWS CLI v1 does not support the `--cli-binary-format raw-in-base64-out` option used by CLI v2.
- Always verify the CLI version when a normally valid command-line option is rejected.

## Engineering Insight

The Lambda configuration was healthy even when the first invocation command failed. The error came from the local AWS CLI client rather than from Lambda itself.

This reinforces the troubleshooting principle:

**Separate application/resource failures from tooling/client failures before changing the infrastructure.**

## Result

The Lambda function `devops-lambda-cli` was successfully deployed and invoked using the AWS CLI.

**Task Status:** ✅ Completed
