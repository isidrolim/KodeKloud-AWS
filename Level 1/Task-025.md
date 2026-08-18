# Enable S3 Bucket Versioning via AWS CLI

## Problem Statement

Enable **versioning** on an existing Amazon S3 bucket:

- **Bucket Name:** `datacenter-s3-27410`
- **Region:** `us-east-1`
- **Required Versioning Status:** `Enabled`

## Steps

1. Verify the AWS CLI environment.
   ```bash
   aws sts get-caller-identity
   aws configure list
   aws configure get region
   ```

2. Verify the S3 bucket exists.
   ```bash
   aws s3 ls
   ```

3. Check the current versioning status.
   ```bash
   aws s3api get-bucket-versioning \
     --bucket datacenter-s3-27410
   ```

   An empty response means versioning has never been enabled.

4. Enable S3 versioning.
   ```bash
   aws s3api put-bucket-versioning \
     --bucket datacenter-s3-27410 \
     --versioning-configuration Status=Enabled
   ```

## Validation

Verify the bucket versioning status.

```bash
aws s3api get-bucket-versioning \
  --bucket datacenter-s3-27410
```

Expected result:

```json
{
    "Status": "Enabled"
}
```

## Important AWS CLI Commands Learned

- `aws s3 ls` – List S3 buckets.
- `get-bucket-versioning` – Check the current versioning configuration.
- `put-bucket-versioning` – Configure versioning on an S3 bucket.
- Empty versioning output → versioning has never been enabled.
- `Enabled` → S3 retains multiple versions of objects.
- `Suspended` → New object versions are no longer being created normally.
- S3 versioning helps recover from accidental object deletion or overwrites.
