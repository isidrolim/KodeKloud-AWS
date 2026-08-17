# Create Public S3 Bucket via AWS CLI

## Problem Statement

Create an Amazon S3 bucket with the following requirements:

- **Bucket Name:** `devops-s3-414457408`
- **Region:** `us-east-1`
- **Access:** Public

> **Note:** New S3 buckets have Block Public Access enabled by default. To allow public access, the required Public Access Block settings must be disabled.

## Steps

1. Verify the AWS CLI environment.
   ```bash
   aws sts get-caller-identity
   aws configure list
   aws configure get region
   ```

2. Check whether the bucket already exists.
   ```bash
   aws s3api list-buckets \
     --query "Buckets[*].Name" \
     --output table
   ```

3. Create the S3 bucket.
   ```bash
   aws s3 mb s3://devops-s3-414457408 \
     --region us-east-1
   ```

4. Check the current Public Access Block configuration.
   ```bash
   aws s3api get-public-access-block \
     --bucket devops-s3-414457408
   ```

5. Disable Block Public Access for the bucket.
   ```bash
   aws s3api put-public-access-block \
     --bucket devops-s3-414457408 \
     --public-access-block-configuration \
     "BlockPublicAcls=false,IgnorePublicAcls=false,BlockPublicPolicy=false,RestrictPublicBuckets=false"
   ```

## Validation

Verify the Public Access Block configuration.

```bash
aws s3api get-public-access-block \
  --bucket devops-s3-414457408
```

Expected result:

```json
{
    "PublicAccessBlockConfiguration": {
        "BlockPublicAcls": false,
        "IgnorePublicAcls": false,
        "BlockPublicPolicy": false,
        "RestrictPublicBuckets": false
    }
}
```

## Important AWS CLI Commands Learned

- `aws s3 mb` – Create an S3 bucket.
- `get-public-access-block` – Check the bucket's Public Access Block configuration.
- `put-public-access-block` – Modify the bucket's Public Access Block settings.
- `true` → public access is blocked by that control.
- `false` → that particular public-access protection is disabled.
- Disabling Block Public Access **allows** public policies/ACLs to be configured; actual anonymous public access still depends on an appropriate bucket policy or ACL.
