# Create VPC with Amazon-Provided IPv6 CIDR via AWS CLI

## Scenario

The Nautilus DevOps team requires a new VPC that supports IPv6 networking using an IPv6 CIDR block allocated directly by AWS.

## Requirement

Create a VPC with:

- **VPC Name:** `xfusion-vpc`
- **Region:** `us-east-1`
- **IPv4 CIDR:** `10.0.0.0/16`
- **IPv6 CIDR:** Amazon-provided

## Initial State

Verify the AWS environment first.

```bash
aws sts get-caller-identity
aws configure list
aws configure get region
```

Inspect the existing VPCs and their IPv4 CIDRs.

```bash
aws ec2 describe-vpcs \
  --query "Vpcs[*].[VpcId,CidrBlock,Tags[?Key=='Name']|[0].Value]" \
  --output table \
  --region us-east-1
```

This helps ensure the selected IPv4 CIDR does not overlap with an existing VPC.

## Troubleshooting Path

The required resource is a standard Amazon VPC, so the correct AWS CLI service is:

```text
aws ec2
```

Discover the VPC creation command:

```bash
aws ec2 help | grep -i vpc
```

Then inspect its documentation:

```bash
aws ec2 create-vpc help | less
```

Search the help page for IPv6:

```text
/amazon-provided
```

The relevant option is:

```text
--amazon-provided-ipv6-cidr-block
```

## Verification Before Fix

The important discovery is that an IPv6-enabled VPC still has a primary IPv4 CIDR.

The command therefore requires:

```text
IPv4 CIDR
    +
Amazon-provided IPv6 CIDR request
    +
Name tag
```

AWS selects the IPv6 CIDR automatically; we do not manually specify the IPv6 network.

## Systematic Elimination

Do not use:

```bash
aws ec2 create-vpc --ipv6-cidr-block ...
```

when the requirement specifically asks for an **Amazon-provided IPv6 CIDR**.

Instead use:

```text
--amazon-provided-ipv6-cidr-block
```

Also remember:

```text
xfusion-vpc
```

is not the actual VPC ID. It is the value of the `Name` tag.

AWS generates the real identifier:

```text
vpc-xxxxxxxxxxxxxxxxx
```

## First Finding

The correct command family is:

```bash
aws ec2 create-vpc
```

and AWS CLI provides a dedicated boolean option to request an Amazon-owned IPv6 range:

```text
--amazon-provided-ipv6-cidr-block
```

## Fix

Create the VPC:

```bash
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --amazon-provided-ipv6-cidr-block \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=xfusion-vpc}]' \
  --region us-east-1
```

AWS then creates:

```text
IPv4 CIDR → 10.0.0.0/16
IPv6 CIDR → Allocated automatically by AWS
Name      → xfusion-vpc
```

## Validation

Query the VPC using its Name tag:

```bash
aws ec2 describe-vpcs \
  --filters "Name=tag:Name,Values=xfusion-vpc" \
  --query "Vpcs[*].[VpcId,State,CidrBlock,Ipv6CidrBlockAssociationSet[*].Ipv6CidrBlock]" \
  --output table \
  --region us-east-1
```

Verify:

```text
Name:       xfusion-vpc
State:      available
IPv4 CIDR:  10.0.0.0/16
IPv6 CIDR:  Amazon-assigned IPv6 range
```

## Lessons Learned

- Standard VPC operations are managed through `aws ec2`.
- `--cidr-block` specifies the primary IPv4 CIDR.
- `--amazon-provided-ipv6-cidr-block` asks AWS to allocate the IPv6 range.
- An IPv6-enabled VPC still has an IPv4 CIDR.
- The VPC's friendly name is stored as a `Name` tag.
- AWS generates the actual `vpc-...` resource identifier.

## Engineering Insight

There is an important difference between:

```text
You provide the network
-----------------------
--cidr-block 10.0.0.0/16
        ↓
IPv4 CIDR chosen by us
```

and:

```text
AWS provides the network
------------------------
--amazon-provided-ipv6-cidr-block
        ↓
IPv6 CIDR allocated by AWS
```

The reusable workflow is:

```text
Understand Requirement
        ↓
Inspect Existing Networks
        ↓
Check for CIDR Overlap
        ↓
Use AWS CLI Help
        ↓
Discover Correct IPv6 Option
        ↓
Create VPC
        ↓
Validate IPv4 + IPv6
```

## Knowledge Check

1. Why does an IPv6-enabled VPC still require an IPv4 CIDR in this creation flow?
2. What is the difference between `--cidr-block` and `--amazon-provided-ipv6-cidr-block`?
3. Who chooses the IPv6 CIDR when using `--amazon-provided-ipv6-cidr-block`?
4. Is `xfusion-vpc` the actual AWS VPC ID or a tag?
5. Why should existing network CIDRs be inspected before creating a new VPC?
