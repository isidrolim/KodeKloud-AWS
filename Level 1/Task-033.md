# Create a VPC via AWS CLI

## Problem Statement

Create a new Amazon VPC with the following requirements:

- **VPC Name:** `nautilus-vpc`
- **Region:** `us-east-1`
- **IPv4 CIDR:** Any valid IPv4 CIDR block
- **Selected CIDR:** `10.0.0.0/24`

## Initial AWS Checks

Verify the AWS identity, configuration, and Region before making changes.

```bash
aws sts get-caller-identity
aws configure list
aws configure get region
```

## Command Discovery

Initially, `vpc-lattice` appeared to be related to VPC creation:

```bash
aws vpc-lattice
```

However, **VPC Lattice is a separate AWS networking service** and does not create standard VPCs.

Searching the EC2 commands revealed the correct operation:

```bash
aws ec2 help | grep -i vpc
```

The required command is:

```text
create-vpc
```

Use AWS CLI help to inspect its syntax:

```bash
aws ec2 create-vpc help | less
```

Useful options discovered:

```text
--cidr-block
--tag-specifications
```

## Create the VPC

Create the VPC using the private IPv4 CIDR `10.0.0.0/24` and assign the required `Name` tag during creation:

```bash
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/24 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=nautilus-vpc}]' \
  --region us-east-1
```

AWS automatically generates the actual VPC ID:

```text
vpc-xxxxxxxxxxxxxxxxx
```

The human-readable name `nautilus-vpc` is stored as:

```text
Key   = Name
Value = nautilus-vpc
```

## Validation

Locate the newly created VPC using its Name tag:

```bash
aws ec2 describe-vpcs \
  --filters "Name=tag:Name,Values=nautilus-vpc" \
  --query "Vpcs[*].[VpcId,State,CidrBlock,IsDefault]" \
  --output table \
  --region us-east-1
```

Expected result:

```text
VpcId:     vpc-xxxxxxxxxxxxxxxxx
State:     available
CIDR:      10.0.0.0/24
IsDefault: False
```

## Important AWS CLI Commands Learned

- `aws ec2 describe-vpcs` – List or inspect VPCs.
- `aws ec2 create-vpc` – Create a standard Amazon VPC.
- `--cidr-block` – Define the IPv4 address range assigned to the VPC.
- `--tag-specifications` – Apply tags while creating the resource.
- `Name` is a **tag**, not the VPC's actual AWS resource identifier.
- AWS automatically generates the real VPC ID.
- `aws vpc-lattice` belongs to **VPC Lattice** and is not used to create a normal VPC.

## Engineering Insight

The AWS CLI service name is not always obvious from the AWS resource name.

```text
Amazon VPC
    ↓
Managed through EC2 API
    ↓
aws ec2 create-vpc
```

Whereas:

```text
AWS VPC Lattice
    ↓
Different networking service
    ↓
aws vpc-lattice
```

For production networks, do not randomly choose a private CIDR simply because it is technically valid.

Before selecting the VPC CIDR, check for overlap with:

```text
Existing VPCs
On-premises networks
VPN-connected networks
Transit Gateway networks
Peered VPCs
Future subnet requirements
```

The troubleshooting pattern remains:

```text
Understand the required resource
        ↓
Find the correct AWS service
        ↓
Use aws <service> help
        ↓
Find the operation
        ↓
Use aws <service> <operation> help
        ↓
Understand required parameters
        ↓
Create the resource
        ↓
Validate against the original requirement
```
