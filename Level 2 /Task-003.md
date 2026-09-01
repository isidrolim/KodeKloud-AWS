# Creating and Launching EC2 Instances from Custom AMIs

## Scenario
The Nautilus DevOps team required an Amazon Machine Image (AMI) of an existing EC2 instance for backup and scaling purposes.

The existing EC2 instance was:

- `datacenter-ec2`

## Requirement
1. Create a custom AMI from the existing `datacenter-ec2` instance.
2. Name the AMI:
   - `datacenter-ec2-ami`
3. Launch a new EC2 instance from the newly created AMI.
4. Name the new instance:
   - `datacenter-ec2-new`
5. Perform all operations in the `us-east-1` region.

## Steps

### 1. Locate the Existing EC2 Instance
Navigate to:

`EC2 → Instances`

Locate the existing instance:

`datacenter-ec2`

### 2. Create the Custom AMI
Select `datacenter-ec2` and choose:

`Actions → Image and templates → Create image`

Set the image name to:

`datacenter-ec2-ami`

Create the image.

### 3. Verify the AMI
Navigate to:

`EC2 → AMIs`

Confirm that:

`datacenter-ec2-ami`

reaches the `Available` state before attempting to launch another instance from it.

### 4. Launch an EC2 Instance from the AMI
Select:

`datacenter-ec2-ami`

Then choose:

`Launch instance from AMI`

Set the new EC2 instance name to:

`datacenter-ec2-new`

Configure the required instance settings and launch the instance.

## Validation
Verify the following:

- `datacenter-ec2` remains available as the source instance.
- Custom AMI `datacenter-ec2-ami` exists.
- The AMI status is `Available`.
- New EC2 instance `datacenter-ec2-new` was created from the custom AMI.
- The new instance reaches the `Running` state.
- All resources were created in `us-east-1`.

## Result
The custom AMI was successfully created from the existing EC2 instance and used to launch a new EC2 instance.

**Task Status:** ✅ Completed

## Key Takeaway
An AMI acts as a reusable template for an EC2 instance. It captures the information required to launch additional instances with the same base system configuration, making AMIs useful for backups, recovery, cloning, and horizontal scaling.

The basic workflow is:

`Existing EC2 → Create AMI → Wait for AMI Available → Launch New EC2 from AMI`
