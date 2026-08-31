# Setting Up an EC2 Instance with an Elastic IP for Application Hosting

## Scenario

The Nautilus DevOps team received a request to deploy a new Linux EC2 instance for application hosting.

Because the application requires a stable public IP address, an Elastic IP (EIP) must be allocated and associated with the new EC2 instance.

## Requirement

Create the following resources in `us-east-1`:

- EC2 instance name: `nautilus-ec2`
- Operating system: Any Linux AMI, such as Ubuntu
- Instance type: `t2.micro`
- Elastic IP name: `nautilus-eip`
- Associate the Elastic IP with `nautilus-ec2`

## Initial State

Verify the AWS identity and configured region before creating resources.

```bash
aws sts get-caller-identity

aws configure list

aws configure get region
```

Expected region:

```text
us-east-1
```

## Troubleshooting Path

Build the dependency path before provisioning:

```text
AWS Account / Region
        ↓
Linux AMI
        ↓
VPC / Subnet
        ↓
Security Group
        ↓
Create EC2 Instance
        ↓
Wait for running
        ↓
Allocate Elastic IP
        ↓
Tag EIP as nautilus-eip
        ↓
Associate EIP with EC2
        ↓
Validate Instance + EIP
```

## Verification Before Fix

Verify that an instance named `nautilus-ec2` does not already exist:

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=nautilus-ec2" \
  --query "Reservations[*].Instances[*].[InstanceId,State.Name,InstanceType,PublicIpAddress]" \
  --output table \
  --region us-east-1
```

Check for an existing Elastic IP:

```bash
aws ec2 describe-addresses \
  --filters "Name=tag:Name,Values=nautilus-eip" \
  --query "Addresses[*].[AllocationId,PublicIp,InstanceId,AssociationId]" \
  --output table \
  --region us-east-1
```

## Systematic Elimination

Before launching the instance, confirm the required dependencies:

```text
Linux AMI available?
        ↓
t2.micro supported?
        ↓
Valid subnet available?
        ↓
Security group available?
        ↓
EC2 instance absent?
        ↓
Safe to create
```

An Elastic IP should not be associated until the EC2 instance exists and its exact instance ID has been identified.

## First Finding

The task requires two separate AWS resources:

```text
EC2 Instance
    nautilus-ec2

Elastic IP
    nautilus-eip
```

The Elastic IP is then associated with the EC2 instance.

This is important because an Elastic IP is not simply a normal automatically assigned EC2 public IP.

```text
Normal Public IPv4
        ↓
May change after stop/start

Elastic IP
        ↓
Allocated to AWS account
        ↓
Can be explicitly associated
        ↓
Provides stable public IPv4 address
```

## Fix

### 1. Create the EC2 instance

After identifying an appropriate Linux AMI, subnet, and security group:

```bash
aws ec2 run-instances \
  --image-id <LINUX-AMI-ID> \
  --instance-type t2.micro \
  --subnet-id <SUBNET-ID> \
  --security-group-ids <SECURITY-GROUP-ID> \
  --tag-specifications \
    'ResourceType=instance,Tags=[{Key=Name,Value=nautilus-ec2}]' \
  --region us-east-1
```

Capture the instance ID instead of repeatedly typing it manually:

```bash
INSTANCE_ID=$(aws ec2 describe-instances \
  --filters \
    "Name=tag:Name,Values=nautilus-ec2" \
    "Name=instance-state-name,Values=pending,running" \
  --query "Reservations[0].Instances[0].InstanceId" \
  --output text \
  --region us-east-1)
```

Verify:

```bash
echo "$INSTANCE_ID"
```

Wait until the instance is running:

```bash
aws ec2 wait instance-running \
  --instance-ids "$INSTANCE_ID" \
  --region us-east-1
```

### 2. Allocate an Elastic IP

Allocate a VPC Elastic IP:

```bash
aws ec2 allocate-address \
  --domain vpc \
  --region us-east-1
```

The response provides values such as:

```text
PublicIp
AllocationId
```

Capture the allocation ID:

```bash
ALLOCATION_ID=$(aws ec2 describe-addresses \
  --query "Addresses[?AssociationId==null] | [-1].AllocationId" \
  --output text \
  --region us-east-1)
```

### 3. Tag the Elastic IP

Apply the required Name tag:

```bash
aws ec2 create-tags \
  --resources "$ALLOCATION_ID" \
  --tags Key=Name,Value=nautilus-eip \
  --region us-east-1
```

A safer approach after tagging is to rediscover it by its required Name:

```bash
ALLOCATION_ID=$(aws ec2 describe-addresses \
  --filters "Name=tag:Name,Values=nautilus-eip" \
  --query "Addresses[0].AllocationId" \
  --output text \
  --region us-east-1)
```

Verify:

```bash
echo "$ALLOCATION_ID"
```

### 4. Associate the Elastic IP

Associate the allocated EIP with the EC2 instance:

```bash
aws ec2 associate-address \
  --instance-id "$INSTANCE_ID" \
  --allocation-id "$ALLOCATION_ID" \
  --region us-east-1
```

AWS returns an association ID similar to:

```text
eipassoc-xxxxxxxxxxxxxxxxx
```

## Validation

Validate the EC2 instance:

```bash
aws ec2 describe-instances \
  --instance-ids "$INSTANCE_ID" \
  --query "Reservations[0].Instances[0].[Tags[?Key=='Name']|[0].Value,InstanceId,State.Name,InstanceType,PublicIpAddress]" \
  --output table \
  --region us-east-1
```

Expected properties:

```text
Name          nautilus-ec2
State         running
InstanceType  t2.micro
PublicIp      <Elastic-IP>
```

Validate the Elastic IP independently:

```bash
aws ec2 describe-addresses \
  --filters "Name=tag:Name,Values=nautilus-eip" \
  --query "Addresses[*].[PublicIp,AllocationId,AssociationId,InstanceId]" \
  --output table \
  --region us-east-1
```

The final dependency should look like:

```text
nautilus-eip
     │
     │ Elastic IP
     ▼
nautilus-ec2
     │
     ├── Linux
     ├── t2.micro
     └── running
```

## Lessons Learned

- An Elastic IP is a separately allocated AWS resource.
- `allocate-address` allocates the Elastic IP.
- `--domain vpc` allocates the EIP for use with VPC resources.
- `create-tags` can assign the required `Name` tag to the EIP.
- `associate-address` associates the Elastic IP with an EC2 instance.
- `AllocationId` identifies the allocated EIP.
- `AssociationId` identifies the EIP-to-resource association.
- Shell variables reduce mistakes when working with long AWS resource IDs.
- Resource creation and association should be validated independently.
- A normal automatically assigned EC2 public IP and an Elastic IP are not the same thing.

## Engineering Insight

The important architecture is:

```text
Internet
   │
   ▼
Elastic IP
(stable public IPv4)
   │
   ▼
EC2 Network Interface
   │
   ▼
nautilus-ec2
```

The EIP lifecycle is independent from the EC2 lifecycle:

```text
ALLOCATE
    ↓
TAG
    ↓
ASSOCIATE
    ↓
VALIDATE
```

This also means that in production an unused allocated Elastic IP should be reviewed and released when no longer required.

The strongest validation is not simply:

```text
"Does nautilus-ec2 exist?"
```

Instead verify:

```text
Does nautilus-ec2 exist?
        ↓
Is it running?
        ↓
Is it t2.micro?
        ↓
Does nautilus-eip exist?
        ↓
Is the EIP associated?
        ↓
Is it associated with the correct instance?
```

## Knowledge Check

1. What is the difference between an automatically assigned EC2 public IP and an Elastic IP?
2. What does `aws ec2 allocate-address` do?
3. Why is `--domain vpc` used when allocating the Elastic IP?
4. What is the difference between an EIP `AllocationId` and `AssociationId`?
5. Which AWS CLI command associates an Elastic IP with an EC2 instance?
6. Why is it useful to store the instance ID and allocation ID in shell variables?
7. What should happen to an Elastic IP when an application is permanently decommissioned?
8. Why should the EC2 instance and Elastic IP be validated independently after the association?
