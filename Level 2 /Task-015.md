# AWS Level 2 – Task 015: Create a Lambda Function

## Scenario
The Nautilus DevOps team is introducing serverless architecture using AWS Lambda. The requirement is to deploy a simple Python Lambda function that returns a custom greeting and HTTP-style status code.

## Requirements

- **Region:** `us-east-1`
- **Function Name:** `xfusion-lambda`
- **Runtime:** Python
- **IAM Role:** `lambda_execution_role`
- **Status Code:** `200`
- **Response Body:** `Welcome to KKE AWS Labs!`
- Complete the task using the AWS Console

## Steps

### 1. Create the Lambda Function

Go to:

`AWS Console → Lambda → Functions → Create function`

Select:

`Author from scratch`

Configure:

- **Function name:** `xfusion-lambda`
- **Runtime:** Python
- **Architecture:** `x86_64`

Create the function.

---

### 2. Create the Required IAM Role

The required `lambda_execution_role` did not initially exist.

Go to:

`IAM → Roles → Create role`

Configure:

- **Trusted entity:** AWS service
- **Use case:** Lambda
- **Permission:** `AWSLambdaBasicExecutionRole`
- **Role name:** `lambda_execution_role`

Create the role.

---

### 3. Attach the IAM Role to Lambda

Go to:

`Lambda → xfusion-lambda → Configuration → Permissions`

Under **Execution role**, select:

`Edit → Use an existing role`

Choose:

`lambda_execution_role`

Save the configuration.

---

### 4. Configure the Lambda Code

Go to:

`Lambda → xfusion-lambda → Code`

Replace the default code in `lambda_function.py` with:

```python
def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "body": "Welcome to KKE AWS Labs!"
    }
```

The console should indicate that the changes are:

`Undeployed`

Click:

`Deploy`

Verify:

`Successfully updated the function "xfusion-lambda".`

---

### 5. Test the Lambda Function

Create a test event named:

`test-event`

Use the following JSON:

```json
{}
```

Run the test.

## Validation

The Lambda execution completed successfully and returned:

```json
{
    "statusCode": 200,
    "body": "Welcome to KKE AWS Labs!"
}
```

Verified:

- **Function:** `xfusion-lambda` ✅
- **Runtime:** Python ✅
- **IAM Role:** `lambda_execution_role` ✅
- **Execution:** Succeeded ✅
- **Status Code:** `200` ✅
- **Response:** `Welcome to KKE AWS Labs!` ✅

The execution flow is:

`Test Event → xfusion-lambda → Python Handler → HTTP 200 + Greeting`

## Lessons Learned

- A Lambda function requires an execution role that Lambda can assume.
- The required IAM role name must match the task specification exactly.
- Editing Lambda code does not automatically deploy the changes; the code must be explicitly deployed.
- A successful invocation alone is not enough validation—the returned payload must also be checked against the requirements.

## Engineering Insight

During the task, the Lambda initially used an automatically generated execution role:

`xfusion-lambda-role-...`

Although this was a valid IAM role, it did not satisfy the requirement for:

`lambda_execution_role`

Rather than assuming the existing role was correct, the configuration was verified against the original requirement, the required role was created, and the Lambda was updated to use it.

This reinforces an important engineering principle:

**A configuration can be technically functional while still being incorrect relative to the required specification.**

## Result

The `xfusion-lambda` function was successfully deployed using Python and the required `lambda_execution_role`.

Testing confirmed that the function executes successfully and returns:

`200 → Welcome to KKE AWS Labs!`

**Task Status:** ✅ Completed
