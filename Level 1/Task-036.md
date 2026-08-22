# Delete a VPC via AWS CLI

## Scenario

The Nautilus DevOps team is cleaning up AWS resources that are no longer required. An existing VPC named `devops-vpc` must be permanently removed.

## Requirement

- **VPC Name:** `devops-vpc`
- **Region:** `us-east-1`
- **Required Final State:** VPC no longer exists

## Initial State

Verify the AWS CLI environment.

```bash
aws sts get-caller-identity
aws configure list
aws configure get region
```

## Troubleshooting Path

For a destructive operation, first identify the exact VPC rather than deleting based only on an assumed resource ID.

```text
Name Tag: devops-vpc
        ↓
Find VPC ID
        ↓
Confirm correct resource
        ↓
Check dependencies if necessary
        ↓
Delete VPC
        ↓
Verify it no longer exists
```

## Verification Before Fix

Locate the VPC using its Name tag.

```bash
aws ec2 describe-vpcs \
  --filters "Name=tag:Name,Values=devops-vpc" \
  --query "Vpcs[*].[VpcId,State,CidrBlock]" \
  --output table \
  --region us-east-1
```

The target VPC was identified as:

```text
Name:  devops-vpc
CIDR:  10.0.0.0/16
State: available
```

AWS assigns the actual VPC identifier in the format:

```text
vpc-xxxxxxxxxxxxxxxxx
```

## Systematic Elimination

Before deleting a production VPC, consider whether resources depend on it.

Possible dependencies include:

```text
VPC
├── Subnets
├── Internet Gateway
├── NAT Gateway
├── Elastic Network Interfaces
├── VPC Endpoints
├── Peering Connections
└── Other networking resources
```

If `delete-vpc` returns a dependency violation, inspect these resources before removing anything.

## First Finding

The VPC existed and its exact VPC ID was successfully identified.

The correct AWS CLI operation was discovered as:

```bash
aws ec2 delete-vpc
```

If syntax is uncertain:

```bash
aws ec2 delete-vpc help
```

## Fix

Delete the identified VPC.

```bash
aws ec2 delete-vpc \
  --vpc-id <vpc-id> \
  --region us-east-1
```

The command completed without returning an error.

## Validation

Verify using the original Name tag:

```bash
aws ec2 describe-vpcs \
  --filters "Name=tag:Name,Values=devops-vpc" \
  --query "Vpcs[*].[VpcId,State,CidrBlock]" \
  --output table \
  --region us-east-1
```

An empty result confirms that `devops-vpc` no longer exists.

You can also query the exact deleted VPC ID:

```bash
aws ec2 describe-vpcs \
  --vpc-ids <vpc-id> \
  --region us-east-1
```

A `InvalidVpcID.NotFound` response confirms that the VPC ID no longer exists.

## Lessons Learned

- `describe-vpcs` is used to inspect and locate VPCs.
- `delete-vpc` requires the actual AWS-generated VPC ID.
- The human-readable VPC name is stored as a `Name` tag.
- Filtering by the Name tag is safer than manually searching large JSON output.
- A successful delete command should still be followed by validation.
- VPC deletion can fail when dependent networking resources still exist.

## Engineering Insight

For destructive AWS operations, use:

```text
IDENTIFY
   ↓
VERIFY
   ↓
CHECK DEPENDENCIES
   ↓
DELETE
   ↓
VERIFY ABSENCE
```

Do not consider a deletion successful merely because the delete command returned no error.

The original success criterion was:

```text
devops-vpc must not exist
```

Therefore the strongest validation is proving that the resource can no longer be found.

## Knowledge Check

1. Why do we need the VPC ID instead of just `devops-vpc` when using `delete-vpc`?
2. Why is filtering with `Name=tag:Name` safer than reading the complete `describe-vpcs` output?
3. What kinds of resources can prevent a VPC from being deleted?
4. What does `InvalidVpcID.NotFound` mean when it appears during post-deletion validation?
5. Why should a destructive command always be followed by an independent verification?
