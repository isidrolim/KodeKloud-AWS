# Create EC2 Instance with RSA Key Pair in Default VPC via AWS CLI

## Scenario

The Nautilus DevOps team requires a new EC2 instance to be created entirely through the AWS CLI. The instance must use the default VPC, default security group, a specific AMI and instance type, and a newly created RSA key pair.

## Requirement

- **Instance Name:** `devops-ec2`
- **AMI:** `ami-0cd59ecaf368e5ccf`
- **Instance Type:** `t2.micro`
- **Key Pair:** `devops-kp`
- **Key Type:** RSA
- **VPC:** Default VPC
- **Security Group:** Default security group
- **Region:** `us-east-1`
- **Final State:** `running`

## Initial State

Verify the AWS CLI identity, configuration, and Region.

```bash
aws sts get-caller-identity
aws configure list
aws configure get region
```

## Troubleshooting Path

Build the dependency path before launching the instance:

```text
AWS Identity / Region
        ↓
Required AMI
        ↓
RSA Key Pair
        ↓
Default VPC
        ↓
Default Security Group
        ↓
Subnet in Default VPC
        ↓
Launch EC2
        ↓
Wait for Running
        ↓
Validate Configuration
```

## Verification Before Fix

### 1. Check whether the required key pair already exists

```bash
aws ec2 describe-key-pairs \
  --key-names devops-kp \
  --region us-east-1
```

The key pair did not exist, so a new RSA key pair was required.

## Systematic Elimination

### 2. Create the RSA key pair

Use AWS CLI help when necessary:

```bash
aws ec2 create-key-pair help | less
```

Create the key pair and save only the private key material:

```bash
aws ec2 create-key-pair \
  --key-name devops-kp \
  --key-type rsa \
  --query "KeyMaterial" \
  --output text \
  --region us-east-1 > devops-kp.pem
```

Protect the private key:

```bash
chmod 400 devops-kp.pem
```

Verify the local file:

```bash
ls -lah devops-kp.pem
```

Verify the AWS-side key pair:

```bash
aws ec2 describe-key-pairs \
  --key-names devops-kp \
  --region us-east-1
```

### Why `--query "KeyMaterial"`?

`create-key-pair` returns several properties, but the `.pem` file must contain only the private key.

```text
AWS create-key-pair response
        ↓
--query "KeyMaterial"
        ↓
Extract only private key
        ↓
--output text
        ↓
Plain-text key
        ↓
> devops-kp.pem
```

The private key material cannot later be recovered using `describe-key-pairs`, so it must be saved during creation.

## First Finding

### 3. Identify the default VPC

```bash
aws ec2 describe-vpcs \
  --filters "Name=is-default,Values=true" \
  --query "Vpcs[0].[VpcId,CidrBlock]" \
  --output table \
  --region us-east-1
```

Result:

```text
VPC ID: vpc-087f07cceb83a5a07
CIDR:   172.31.0.0/16
```

### 4. Identify the default security group

```bash
aws ec2 describe-security-groups \
  --filters \
    "Name=vpc-id,Values=vpc-087f07cceb83a5a07" \
    "Name=group-name,Values=default" \
  --query "SecurityGroups[0].[GroupId,GroupName,VpcId]" \
  --output table \
  --region us-east-1
```

Result:

```text
Security Group: sg-0d8ccd832d9914d5d
Group Name:     default
VPC:            vpc-087f07cceb83a5a07
```

### 5. Identify available subnets in the default VPC

```bash
aws ec2 describe-subnets \
  --filters "Name=vpc-id,Values=vpc-087f07cceb83a5a07" \
  --query "Subnets[*].[SubnetId,AvailabilityZone,CidrBlock]" \
  --output table \
  --region us-east-1
```

A valid subnet from the default VPC was selected for the instance.

## Fix

Use AWS CLI help to inspect the launch syntax if needed:

```bash
aws ec2 run-instances help | less
```

Useful parameters:

```text
--image-id
--instance-type
--key-name
--subnet-id
--security-group-ids
--tag-specifications
```

Launch the EC2 instance:

```bash
aws ec2 run-instances \
  --image-id ami-0cd59ecaf368e5ccf \
  --instance-type t2.micro \
  --key-name devops-kp \
  --subnet-id <default-vpc-subnet-id> \
  --security-group-ids sg-0d8ccd832d9914d5d \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=devops-ec2}]' \
  --region us-east-1
```

The initial EC2 state may be:

```text
pending
```

This is expected because EC2 provisioning is asynchronous.

Wait for the instance to reach `running`:

```bash
aws ec2 wait instance-running \
  --filters "Name=tag:Name,Values=devops-ec2" \
  --region us-east-1
```

## Validation

Validate more than just the existence of the instance.

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=devops-ec2" \
  --query "Reservations[*].Instances[*].[InstanceId,State.Name,InstanceType,KeyName,SubnetId,VpcId]" \
  --output table \
  --region us-east-1
```

Final validation confirmed:

```text
Instance:       devops-ec2
State:          running
Instance Type:  t2.micro
Key Pair:       devops-kp
VPC:            vpc-087f07cceb83a5a07
Security Group: default
AMI:            ami-0cd59ecaf368e5ccf
```

## Lessons Learned

- An EC2 key pair must exist before `run-instances` can reference it with `--key-name`.
- `create-key-pair` can generate an RSA key pair through AWS CLI.
- `--query "KeyMaterial"` extracts only the private key from the creation response.
- `--output text` removes JSON formatting before writing the key to a `.pem` file.
- Private key material is returned during creation and cannot simply be retrieved later with `describe-key-pairs`.
- `chmod 400` protects the private key from overly permissive filesystem access.
- The default VPC can be located with `Name=is-default,Values=true`.
- Security groups belong to VPCs, so verify the security group belongs to the intended VPC.
- A subnet must also belong to the intended VPC.
- EC2 creation is asynchronous: `pending` → `running`.
- Validation should confirm configuration, not merely resource existence.

## Engineering Insight

EC2 creation is not just:

```text
run-instances
```

It is a dependency chain:

```text
AMI
 │
 ├── Instance Type
 │
 ├── Key Pair
 │
 └── Network
       │
       ├── VPC
       ├── Subnet
       └── Security Group
              ↓
          EC2 Instance
              ↓
           Running
              ↓
      Validate Requirements
```

A stronger validation question is not:

```text
"Does devops-ec2 exist?"
```

but:

```text
Does devops-ec2 exist?
        ↓
Is it running?
        ↓
Is it t2.micro?
        ↓
Does it use devops-kp?
        ↓
Is it in the correct subnet?
        ↓
Is it in the default VPC?
        ↓
Does it have the default security group?
```

## Knowledge Check

1. Why must `devops-kp` exist before launching the EC2 instance?
2. Why did we use `--query "KeyMaterial"` when creating the key pair?
3. Why should the generated `.pem` file be changed to mode `400`?
4. How can you identify the default VPC without knowing its VPC ID beforehand?
5. Why must the subnet and security group belong to the same VPC as the instance?
6. Why can `run-instances` initially return `pending` instead of `running`?
7. Why is checking only whether the EC2 instance exists insufficient validation?
