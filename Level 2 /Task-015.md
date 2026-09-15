# AWS Level 2 – Task 015: Create a Lambda Function

## Scenario
The Nautilus DevOps team is introducing serverless architecture using AWS Lambda. The task is to deploy a simple Python Lambda function that returns a custom greeting with an HTTP-style status code of `200`.

## Requirements

- **Region:** `us-east-1`
- **Lambda Function:** `xfusion-lambda`
- **Runtime:** Python
- **IAM Role:** `lambda_execution_role`
- **Status Code:** `200`
- **Response Body:** `Welcome to KKE AWS Labs!`
- Complete the task using the **AWS Console**

## Steps

### 1. Open AWS Lambda

Go to:

`AWS Console → Lambda → Functions → Create function`

Select:

`Author from scratch`

Configure:

- **Function name:** `xfusion-lambda`
- **Runtime:** Select an available Python runtime
- **Architecture:** `x86_64`

---

### 2. Configure the Existing IAM Role

Expand:

`Change default execution role`

Select:

`Use an existing role`

Choose:

`lambda_execution_role`

Click:

`Create function`

---

### 3. Configure the Lambda Code

Once the function is created, go to the **Code** tab.

Replace the default Python code with:

```python
import json

def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "body": "Welcome to KKE AWS Labs!"
    }
```

Click:

`Deploy`

---

### 4. Create a Test Event

Click:

`Test → Create new event`

Configure:

- **Event name:** `test-event`
- Keep the default JSON event

Example:

```json
{}
```

Click:

`Save`

Then click:

`Test`

---

## Validation

The Lambda execution should complete successfully.

Verify the returned response contains:

```json
{
  "statusCode": 200,
  "body": "Welcome to KKE AWS Labs!"
}
```

Also verify:

- **Function name:** `xfusion-lambda`
- **Runtime:** Python
- **Execution role:** `lambda_execution_role`
- **Execution status:** Succeeded
- **Status code:** `200`
- **Body:** `Welcome to KKE AWS Labs!`

The execution flow is:

```text
Test Event
    ↓
xfusion-lambda
    ↓
Python lambda_handler()
    ↓
statusCode: 200
body: Welcome to KKE AWS Labs!
```

## Result

The `xfusion-lambda` AWS Lambda function was successfully created using the Python runtime and the existing `lambda_execution_role`.

The function was deployed and tested successfully, returning:

`200 → Welcome to KKE AWS Labs!`

**Task Status:** ✅ Completed
