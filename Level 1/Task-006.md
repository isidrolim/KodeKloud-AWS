# Launch EC2 Instance via AWS CLI

## Problem Statement

Launch an **Amazon EC2 Instance** with the following requirements:

- **Instance Name:** `nautilus-ec2`
- **AMI:** Amazon Linux
- **Instance Type:** `t2.micro`
- **Key Pair:** `nautilus-kp` (RSA)
- **Security Group:** Default Security Group
- **Region:** `us-east-1`

## Steps

1. Verify the AWS CLI is authenticated.
   ```bash
   aws sts get-caller-identity
   ```

2. Verify the AWS CLI configuration.
   ```bash
   aws configure list
   ```

3. Verify the target Region.
   ```bash
   aws configure get region
   ```

4. Retrieve the latest Amazon Linux AMI.
   ```bash
   aws ssm get-parameter \
     --name /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64 \
     --query "Parameter.Value" \
     --output text \
     --region us-east-1
   ```

5. Verify the instance type.
   ```bash
   aws ec2 describe-instance-types \
     --instance-types t2.micro \
     --query "InstanceTypes[0].[InstanceType,VCpuInfo.DefaultVCpus,MemoryInfo.SizeInMiB]" \
     --output table \
     --region us-east-1
   ```

6. Check whether the key pair already exists.
   ```bash
   aws ec2 describe-key-pairs \
     --key-names nautilus-kp \
     --region us-east-1
   ```

7. Create the RSA key pair (if it does not exist).
   ```bash
   aws ec2 create-key-pair \
     --key-name nautilus-kp \
     --key-type rsa \
     --query "KeyMaterial" \
     --output text \
     --region us-east-1 > nautilus-kp.pem

   chmod 400 nautilus-kp.pem
   ```

8. Retrieve the default VPC.
   ```bash
   aws ec2 describe-vpcs \
     --filters "Name=is-default,Values=true" \
     --query "Vpcs[0].[VpcId,CidrBlock]" \
     --output table \
     --region us-east-1
   ```

9. Retrieve the default security group.
   ```bash
   aws ec2 describe-security-groups \
     --filters \
       "Name=vpc-id,Values=<vpc-id>" \
       "Name=group-name,Values=default" \
     --query "SecurityGroups[0].[GroupId,GroupName,VpcId]" \
     --output table \
     --region us-east-1
   ```

10. Retrieve the default subnets.
    ```bash
    aws ec2 describe-subnets \
      --filters \
        "Name=vpc-id,Values=<vpc-id>" \
        "Name=default-for-az,Values=true" \
      --query "Subnets[*].[SubnetId,AvailabilityZone,CidrBlock]" \
      --output table \
      --region us-east-1
    ```

11. Launch the EC2 instance.
    ```bash
    aws ec2 run-instances \
      --image-id <ami-id> \
      --instance-type t2.micro \
      --key-name nautilus-kp \
      --security-group-ids <security-group-id> \
      --subnet-id <subnet-id> \
      --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=nautilus-ec2}]' \
      --count 1 \
      --region us-east-1
    ```

## Validation

Verify the EC2 instance.

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=nautilus-ec2" \
  --query "Reservations[*].Instances[*].[InstanceId,State.Name,InstanceType,Placement.AvailabilityZone,KeyName,SubnetId,VpcId]" \
  --output table \
  --region us-east-1
```
