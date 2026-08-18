# Copy File to Existing S3 Bucket via AWS CLI

## Problem Statement

Copy an existing local file to an existing Amazon S3 bucket:

- **Source File:** `/tmp/xfusion.txt`
- **S3 Bucket:** `xfusion-cp-24627`
- **Region:** `us-east-1`

## Steps

1. Verify the AWS CLI environment.
   ```bash
   aws sts get-caller-identity
   aws configure list
   aws configure get region
   ```

2. Verify the local file exists.
   ```bash
   ls -lah /tmp/xfusion.txt
   ```

3. Verify the S3 bucket exists.
   ```bash
   aws s3 ls
   ```

4. Copy the local file to the S3 bucket.
   ```bash
   aws s3 cp \
     /tmp/xfusion.txt \
     s3://xfusion-cp-24627/
   ```

## Validation

Verify the uploaded file exists in the S3 bucket.

```bash
aws s3 ls s3://xfusion-cp-24627/
```

Expected object:

```text
xfusion.txt
```

## Important AWS CLI Commands Learned

- `aws s3 ls` – List S3 buckets or objects.
- `aws s3 cp` – Copy files between local storage and Amazon S3.
- S3 paths use the format `s3://bucket-name/`.
- Basic copy syntax:
  ```bash
  aws s3 cp <source> <destination>
  ```
- Always verify both the **source file** and **destination bucket** before copying.
