# Create Private S3 Bucket via AWS CLI

## Problem Statement

Create an Amazon S3 bucket with the following requirements:

- **Bucket Name:** `devops-s3-13532`
- **Region:** `us-east-1`
- **Public Access:** Block all public access
- **Bucket Type:** Private

## Steps

1. Verify the AWS CLI environment.
   ```bash
   aws sts get-caller-identity
   aws configure list
   aws configure get region
   ```

2. Check existing S3 buckets.
   ```bash
   aws s3api list-buckets \
     --query "Buckets[*].Name" \
     --output table
   ```

3. Create the S3 bucket.
   ```bash
   aws s3 mb s3://devops-s3-13532 \
     --region us-east-1
   ```

4. Verify the bucket was created.
   ```bash
   aws s3api list-buckets \
     --query "Buckets[*].Name" \
     --output table
   ```

5. Check the bucket's Public Access Block configuration.
   ```bash
   aws s3api get-public-access-block \
     --bucket devops-s3-13532
   ```

6. If Block Public Access is not already enabled, enable all four protections.
   ```bash
   aws s3api put-public-access-block \
     --bucket devops-s3-13532 \
     --public-access-block-configuration \
     "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"
   ```

## Validation

Verify that all Public Access Block settings are enabled.

```bash
aws s3api get-public-access-block \
  --bucket devops-s3-13532
```

Expected result:

```json
{
    "PublicAccessBlockConfiguration": {
        "BlockPublicAcls": true,
        "IgnorePublicAcls": true,
        "BlockPublicPolicy": true,
        "RestrictPublicBuckets": true
    }
}
```

## Important AWS CLI Commands Learned

- `aws s3 mb` – Make/create an S3 bucket.
- `aws s3api list-buckets` – List buckets using the S3 API.
- `get-public-access-block` – Inspect the bucket's public-access protection.
- `put-public-access-block` – Configure Block Public Access.
- New S3 buckets have **Block Public Access enabled by default**, but always verify the configuration rather than assuming it.
