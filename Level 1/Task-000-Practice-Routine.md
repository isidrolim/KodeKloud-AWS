# Daily AWS CLI Practice Routine (SSO)

## 1. Login using AWS SSO

Configure SSO (one-time setup).

```bash
aws configure sso
```

Provide the following information:

- SSO Start URL
- SSO Region
- AWS Account
- Permission Set / Role
- Default Region (e.g. ap-southeast-1)
- Default Output (table/json)

Login to AWS.

```bash
aws sso login
```

Verify the active identity.

```bash
aws sts get-caller-identity
```

---

## 2. Verify the AWS CLI Configuration

```bash
aws configure list
```

Verify the current Region.

```bash
aws configure get region
```

---

## 3. Daily EC2 Inventory

List all Windows servers.

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=WS*" \
  --query "Reservations[*].Instances[*].[InstanceId,InstanceType,PrivateIpAddress,State.Name,Tags[?Key=='Name']|[0].Value]" \
  --output text \
  --region ap-southeast-1 | nl
```

Count Windows servers.

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=WS*" \
  --query "Reservations[*].Instances[*].InstanceId" \
  --output text \
  --region ap-southeast-1 | wc -w
```

---

## 4. Explore AWS Resources

List Security Groups.

```bash
aws ec2 describe-security-groups \
  --output table
```

List VPCs.

```bash
aws ec2 describe-vpcs \
  --output table
```

List Subnets.

```bash
aws ec2 describe-subnets \
  --output table
```

List EBS Volumes.

```bash
aws ec2 describe-volumes \
  --output table
```

List Elastic IPs.

```bash
aws ec2 describe-addresses \
  --output table
```

---

## 5. Logout

SSO credentials expire automatically.

To login again:

```bash
aws sso login
```
