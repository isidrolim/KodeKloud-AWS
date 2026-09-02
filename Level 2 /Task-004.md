# AWS Level 2 – Task 4: Configuring Secure SSH Access to an EC2 Instance via AWS CLI

## Scenario

The Nautilus DevOps team needs a new EC2 instance that can be accessed securely from the `aws-client` host using passwordless SSH.

The EC2 instance must:

- Be named `datacenter-ec2`
- Use instance type `t2.micro`
- Be created in `us-east-1`
- Allow SSH access from the `aws-client`
- Use an SSH key named `id_rsa` stored under `/root/.ssh/`
- Allow the `root` user to authenticate using the generated SSH key

The entire AWS portion of the task will be performed using the AWS CLI from `aws-client`.

---

## 1. Verify AWS CLI Access

Confirm that the lab credentials are working:

```bash
aws sts get-caller-identity
```

Set the region for the session:

```bash
export AWS_DEFAULT_REGION=us-east-1
```

Verify:

```bash
aws configure get region
```

---

## 2. Create the SSH Key Pair on `aws-client`

First check whether `/root/.ssh/id_rsa` already exists:

```bash
ls -l /root/.ssh/id_rsa*
```

If the key does not exist, create it:

```bash
mkdir -p /root/.ssh
chmod 700 /root/.ssh

ssh-keygen -t rsa -b 2048 -f /root/.ssh/id_rsa -N ""
```

This creates:

```text
/root/.ssh/id_rsa
/root/.ssh/id_rsa.pub
```

Verify:

```bash
ls -l /root/.ssh/id_rsa*
```

---

## 3. Import the Public Key into AWS

AWS needs the public key so that EC2 can install it on the new instance.

Import `/root/.ssh/id_rsa.pub` as an EC2 key pair:

```bash
aws ec2 import-key-pair \
  --key-name id_rsa \
  --public-key-material fileb:///root/.ssh/id_rsa.pub
```

Verify:

```bash
aws ec2 describe-key-pairs \
  --key-names id_rsa \
  --query 'KeyPairs[*].[KeyName,KeyPairId]' \
  --output table
```

---

## 4. Identify the VPC and Subnet

List the available VPCs:

```bash
aws ec2 describe-vpcs \
  --query 'Vpcs[*].[VpcId,IsDefault,CidrBlock]' \
  --output table
```

Store the default VPC ID:

```bash
VPC_ID=$(aws ec2 describe-vpcs \
  --filters "Name=is-default,Values=true" \
  --query 'Vpcs[0].VpcId' \
  --output text)

echo "$VPC_ID"
```

Now find an available subnet in that VPC:

```bash
aws ec2 describe-subnets \
  --filters "Name=vpc-id,Values=$VPC_ID" \
  --query 'Subnets[*].[SubnetId,AvailabilityZone,CidrBlock,MapPublicIpOnLaunch]' \
  --output table
```

Choose the appropriate subnet and store its ID:

```bash
SUBNET_ID=<subnet-id>
```

Example:

```bash
SUBNET_ID=subnet-xxxxxxxxxxxxxxxxx
```

---

## 5. Identify or Create a Security Group for SSH

Check the existing security groups:

```bash
aws ec2 describe-security-groups \
  --filters "Name=vpc-id,Values=$VPC_ID" \
  --query 'SecurityGroups[*].[GroupId,GroupName]' \
  --output table
```

If a suitable SSH security group does not already exist, create one:

```bash
SG_ID=$(aws ec2 create-security-group \
  --group-name datacenter-ssh-sg \
  --description "SSH access for datacenter-ec2" \
  --vpc-id "$VPC_ID" \
  --query 'GroupId' \
  --output text)

echo "$SG_ID"
```

Allow SSH on TCP port 22.

For this temporary lab environment:

```bash
aws ec2 authorize-security-group-ingress \
  --group-id "$SG_ID" \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0
```

> In production, SSH should not normally be exposed to `0.0.0.0/0`. Restrict port 22 to the trusted management source or use AWS Systems Manager Session Manager.

Verify:

```bash
aws ec2 describe-security-groups \
  --group-ids "$SG_ID" \
  --query 'SecurityGroups[*].IpPermissions' \
  --output json
```

---

## 6. Find an Appropriate AMI

Do not hard-code an AMI ID because AMI IDs are region-specific and can change.

For example, find a current Amazon Linux 2023 AMI:

```bash
AMI_ID=$(aws ssm get-parameter \
  --name /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64 \
  --query 'Parameter.Value' \
  --output text)

echo "$AMI_ID"
```

Verify the AMI:

```bash
aws ec2 describe-images \
  --image-ids "$AMI_ID" \
  --query 'Images[*].[ImageId,Name,Architecture,State]' \
  --output table
```

---

## 7. Prepare User Data to Enable Root SSH

The task specifically requires the generated key to be authorized for the `root` user.

Create a user-data script:

```bash
cat > /tmp/user-data.sh <<'EOF'
#!/bin/bash

mkdir -p /root/.ssh
chmod 700 /root/.ssh

if [ -f /home/ec2-user/.ssh/authorized_keys ]; then
    cp /home/ec2-user/.ssh/authorized_keys /root/.ssh/authorized_keys
    chmod 600 /root/.ssh/authorized_keys
    chown -R root:root /root/.ssh
fi

sed -i 's/^#PermitRootLogin.*/PermitRootLogin prohibit-password/' /etc/ssh/sshd_config
sed -i 's/^PermitRootLogin.*/PermitRootLogin prohibit-password/' /etc/ssh/sshd_config

systemctl restart sshd
EOF
```

Review it before using it:

```bash
cat /tmp/user-data.sh
```

---

## 8. Launch the EC2 Instance

Launch the `t2.micro` instance using the imported `id_rsa` key:

```bash
INSTANCE_ID=$(aws ec2 run-instances \
  --image-id "$AMI_ID" \
  --instance-type t2.micro \
  --key-name id_rsa \
  --subnet-id "$SUBNET_ID" \
  --security-group-ids "$SG_ID" \
  --associate-public-ip-address \
  --user-data file:///tmp/user-data.sh \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=datacenter-ec2}]' \
  --query 'Instances[0].InstanceId' \
  --output text)

echo "$INSTANCE_ID"
```

---

## 9. Wait for the Instance to Start

Instead of repeatedly checking manually:

```bash
aws ec2 wait instance-running \
  --instance-ids "$INSTANCE_ID"
```

Then verify:

```bash
aws ec2 describe-instances \
  --instance-ids "$INSTANCE_ID" \
  --query 'Reservations[*].Instances[*].[InstanceId,Tags[?Key==`Name`]|[0].Value,InstanceType,State.Name,PublicIpAddress]' \
  --output table
```

Expected values should include:

```text
Name          : datacenter-ec2
Instance Type : t2.micro
State         : running
```

---

## 10. Retrieve the Public IP Address

Store the instance public IP:

```bash
PUBLIC_IP=$(aws ec2 describe-instances \
  --instance-ids "$INSTANCE_ID" \
  --query 'Reservations[0].Instances[0].PublicIpAddress' \
  --output text)

echo "$PUBLIC_IP"
```

---

## 11. Wait for EC2 Status Checks

Before troubleshooting SSH, verify that AWS considers the instance healthy:

```bash
aws ec2 wait instance-status-ok \
  --instance-ids "$INSTANCE_ID"
```

Verify:

```bash
aws ec2 describe-instance-status \
  --instance-ids "$INSTANCE_ID" \
  --query 'InstanceStatuses[*].[InstanceId,InstanceState.Name,SystemStatus.Status,InstanceStatus.Status]' \
  --output table
```

The system and instance checks should report:

```text
ok
```

---

## 12. Test Passwordless Root SSH

Ensure the private key has secure permissions:

```bash
chmod 600 /root/.ssh/id_rsa
```

Test SSH:

```bash
ssh -i /root/.ssh/id_rsa root@"$PUBLIC_IP"
```

If prompted to trust the host fingerprint, enter:

```text
yes
```

Successful access should result in a shell on the EC2 instance without asking for a password.

---

## 13. Validate the Final Configuration

From `aws-client`, verify the EC2 configuration:

```bash
aws ec2 describe-instances \
  --instance-ids "$INSTANCE_ID" \
  --query 'Reservations[*].Instances[*].[Tags[?Key==`Name`]|[0].Value,InstanceId,InstanceType,KeyName,State.Name,PublicIpAddress]' \
  --output table
```

Verify the AWS key pair:

```bash
aws ec2 describe-key-pairs \
  --key-names id_rsa \
  --output table
```

Finally verify passwordless root SSH:

```bash
ssh -i /root/.ssh/id_rsa root@"$PUBLIC_IP" "hostname && whoami"
```

Expected result:

```text
<EC2-hostname>
root
```

---

## Success Criteria

The task is complete when:

- EC2 instance `datacenter-ec2` exists.
- Instance type is `t2.micro`.
- Instance is running in `us-east-1`.
- SSH key `/root/.ssh/id_rsa` exists on `aws-client`.
- AWS EC2 key pair `id_rsa` exists.
- The corresponding public key is authorized on the EC2 instance.
- TCP/22 is reachable from the required source.
- `aws-client` can SSH to the EC2 instance as `root`.
- SSH authentication succeeds without requiring a password.

## Engineering Insight

Passwordless SSH depends on several components working together:

```text
aws-client
    │
    ├── Private Key: /root/.ssh/id_rsa
    │
    ▼
Network Connectivity
    │
    ▼
Security Group TCP/22
    │
    ▼
EC2 sshd
    │
    ▼
/root/.ssh/authorized_keys
    │
    ▼
Public/Private Key Match
    │
    ▼
Successful root SSH
```

If SSH fails, troubleshoot this dependency path systematically rather than immediately changing the SSH configuration.

The first question should always be:

**At which layer does the connection first fail?**
