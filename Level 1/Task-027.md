# Copy S3 Bucket Data Locally and Delete Bucket via AWS CLI

## Problem Statement

Copy all contents from an existing S3 bucket to the local `/opt` directory, then delete the S3 bucket.

- **S3 Bucket:** `devops-bck-24653`
- **Destination:** `/opt`
- **Region:** `us-east-1`

## Steps

1. Verify the AWS CLI environment.
   ```bash
   aws sts get-caller-identity
   aws configure list
   aws configure get region
   ```

2. Verify the S3 bucket exists and inspect its contents.
   ```bash
   aws s3 ls s3://devops-bck-24653/ \
     --recursive
   ```

3. Copy all objects from S3 to `/opt`.
   ```bash
   aws s3 cp \
     s3://devops-bck-24653/ \
     /opt/ \
     --recursive
   ```

4. Verify the files were downloaded successfully.
   ```bash
   ls -lah /opt/
   ```

5. Compare with the objects still stored in S3.
   ```bash
   aws s3 ls s3://devops-bck-24653/ \
     --recursive
   ```

6. After confirming the local copy, delete the objects and bucket.
   ```bash
   aws s3 rb s3://devops-bck-24653 \
     --force
   ```

## Validation

Verify the bucket no longer exists.

```bash
aws s3 ls | grep devops-bck-24653
```

No output confirms the bucket was removed.

Verify the downloaded data remains locally.

```bash
ls -lah /opt/
```

## Important AWS CLI Commands Learned

- `aws s3 cp --recursive` – Copy all objects under an S3 bucket or prefix.
- `aws s3 rb` – Remove an S3 bucket.
- `aws s3 rb --force` – Delete the bucket contents first, then remove the bucket.
- `*` is not used like a normal Linux wildcard when copying all S3 objects with `aws s3 cp`; use `--recursive`.
- Always **validate the local copy before deleting the source bucket**.
