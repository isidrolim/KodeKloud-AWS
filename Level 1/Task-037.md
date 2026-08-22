# Create a Private S3 Bucket and Block Public Access via AWS CLI

## Scenario

The Nautilus DevOps team requires a new S3 bucket for data migration. The bucket must remain private, with all forms of public access explicitly blocked.

## Requirement

- **Bucket Name:** `nautilus-s3-26715`
- **Region:** `us-east-1`
- **Access:** Private
- **Block Public Access:** All four settings enabled

## Initial State

Verify the AWS CLI identity, configuration, and Region.

```bash
aws sts get-caller-identity
aws configure list
aws configure get region
```

Check whether the bucket already exists in the account.

```bash
aws s3api list-buckets \
  --query "Buckets[*].Name" \
  --output table
```

The initial result showed no existing `nautilus-s3-26715` bucket.

## Troubleshooting Path

```text
Verify AWS Context
        ↓
Check Existing Buckets
        ↓
Create Bucket
        ↓
Configure Block Public Access
        ↓
Verify Bucket Exists
        ↓
Verify All Four Controls = true
```

## Verification Before Fix

Confirm the exact bucket name before creation:

```text
nautilus-s3-26715
```

S3 bucket names are globally unique, so verifying and using the exact task-provided name is important.

## Systematic Elimination

The high-level AWS S3 command for creating a bucket is:

```text
mb = make bucket
```

The S3 API command family is used for the security configuration:

```text
aws s3api
```

AWS CLI help can be used to discover the required operation:

```bash
aws s3api help | less
```

The relevant operations are:

```text
put-public-access-block
get-public-access-block
```

## First Finding

The required bucket did not exist, so it was safe to create it.

## Fix

Create the S3 bucket:

```bash
aws s3 mb s3://nautilus-s3-26715 \
  --region us-east-1
```

Verify creation:

```bash
aws s3api list-buckets \
  --query "Buckets[*].Name" \
  --output table
```

Explicitly configure all four S3 Block Public Access controls:

```bash
aws s3api put-public-access-block \
  --bucket nautilus-s3-26715 \
  --public-access-block-configuration \
  "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"
```

## Validation

Verify the Public Access Block configuration:

```bash
aws s3api get-public-access-block \
  --bucket nautilus-s3-26715
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

All four values must be:

```text
true
```

Final state:

```text
Bucket:                 nautilus-s3-26715
BlockPublicAcls:        true
IgnorePublicAcls:       true
BlockPublicPolicy:      true
RestrictPublicBuckets:  true
Result:                 Private / Public Access Blocked
```

## Lessons Learned

- `aws s3 mb` creates an S3 bucket.
- `aws s3api` provides lower-level configuration operations.
- `put-public-access-block` configures S3 public-access protection.
- `get-public-access-block` verifies the current configuration.
- Never rely only on the creation command succeeding; independently validate the security configuration.
- Carefully verify resource names when troubleshooting `NoSuchBucket` errors.

## Engineering Insight

S3 Block Public Access consists of four separate controls:

```text
BlockPublicAcls
    ↓
Prevents new public ACLs

IgnorePublicAcls
    ↓
Ignores public permissions granted through ACLs

BlockPublicPolicy
    ↓
Prevents public bucket policies

RestrictPublicBuckets
    ↓
Restricts access when a public policy exists
```

For this requirement:

```text
Private S3 Bucket
        ↓
BlockPublicAcls       = true
IgnorePublicAcls      = true
BlockPublicPolicy     = true
RestrictPublicBuckets = true
```

The reusable workflow is:

```text
INSPECT
   ↓
CREATE
   ↓
SECURE
   ↓
VERIFY
```

## Knowledge Check

1. What does `mb` mean in `aws s3 mb`?
2. What is the difference between `aws s3` and `aws s3api`?
3. Why are there four separate S3 Block Public Access controls?
4. What does `NoSuchBucket` tell you when running `get-public-access-block`?
5. Why should the security configuration be verified after bucket creation?
