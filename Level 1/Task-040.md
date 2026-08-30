# Terminate an EC2 Instance via AWS CLI

## Scenario

The Nautilus DevOps team identified an obsolete EC2 instance that is no longer required. The instance named `devops-ec2` must be permanently terminated using AWS CLI.

## Requirement

- **Instance Name:** `devops-ec2`
- **Region:** `us-east-1`
- **Action:** Terminate the EC2 instance
- **Required Final State:** `terminated`

## Initial State

Verify the AWS environment first.

```bash
aws sts get-caller-identity
aws configure list
aws configure get region
```

## Troubleshooting Path

Because termination is destructive, identify the exact resource before making any change.

```text
Find devops-ec2
      ↓
Get Instance ID
      ↓
Verify current state
      ↓
Terminate exact Instance ID
      ↓
Wait for terminated
      ↓
Validate final state
```

## Verification Before Fix

Locate `devops-ec2` using its Name tag.

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=devops-ec2" \
  --query "Reservations[*].Instances[*].[InstanceId,State.Name,InstanceType]" \
  --output table \
  --region us-east-1
```

This provides the AWS-generated instance ID:

```text
i-xxxxxxxxxxxxxxxxx
```

The instance ID, rather than the Name tag, is used for the termination operation.

## Systematic Elimination

Before terminating an instance, verify:

```text
Correct Name?
    ↓
Correct Instance ID?
    ↓
Correct Region?
    ↓
Correct current state?
    ↓
Safe to permanently terminate?
```

Do not terminate an instance based on an assumed or previously recorded instance ID.

## First Finding

The target `devops-ec2` instance existed and its exact instance ID was identified using `describe-instances`.

The required AWS CLI operation is:

```bash
aws ec2 terminate-instances
```

If the syntax is unfamiliar:

```bash
aws ec2 terminate-instances help | less
```

## Fix

Terminate the identified instance:

```bash
aws ec2 terminate-instances \
  --instance-ids <instance-id> \
  --region us-east-1
```

AWS reports a state transition similar to:

```text
PreviousState: running
CurrentState:  shutting-down
```

Termination is asynchronous, so the instance may not immediately show `terminated`.

Wait for AWS to complete the operation:

```bash
aws ec2 wait instance-terminated \
  --instance-ids <instance-id> \
  --region us-east-1
```

## Validation

Verify the final state:

```bash
aws ec2 describe-instances \
  --instance-ids <instance-id> \
  --query "Reservations[*].Instances[*].[InstanceId,State.Name]" \
  --output table \
  --region us-east-1
```

Expected result:

```text
i-xxxxxxxxxxxxxxxxx
terminated
```

Final lifecycle:

```text
running
   ↓
terminate-instances
   ↓
shutting-down
   ↓
terminated
```

The task is complete only after the instance reaches:

```text
State = terminated
```

## Lessons Learned

- `describe-instances` should be used first to identify the exact target.
- EC2 Name values are tags; destructive operations normally use the actual `i-...` instance ID.
- `terminate-instances` permanently terminates an EC2 instance.
- Termination is asynchronous.
- `aws ec2 wait instance-terminated` avoids repeatedly polling the instance manually.
- A successful termination request does not by itself prove that termination has completed.
- Always validate the final state against the original requirement.

## Engineering Insight

For destructive AWS operations, follow:

```text
IDENTIFY
   ↓
VERIFY
   ↓
UNDERSTAND IMPACT
   ↓
EXECUTE
   ↓
WAIT
   ↓
VALIDATE
```

There is an important difference between:

```text
stop-instances
```

and:

```text
terminate-instances
```

Stopping preserves the EC2 instance so it can normally be started again.

Termination permanently removes the instance and can also delete EBS volumes configured with:

```text
DeleteOnTermination = true
```

Therefore, in production, the dependency path should include:

```text
EC2 Instance
    │
    ├── EBS volumes
    ├── Elastic IP
    ├── ENIs
    ├── DNS records
    ├── Load balancer targets
    ├── Auto Scaling membership
    └── Application dependencies
```

before approving termination.

## Knowledge Check

1. Why did we run `describe-instances` before `terminate-instances`?
2. Why do we terminate using the instance ID instead of only the `Name` tag?
3. What is the difference between `stopped` and `terminated`?
4. Why might an instance show `shutting-down` immediately after the termination command?
5. What does `aws ec2 wait instance-terminated` do?
6. What can happen to attached EBS volumes when an EC2 instance is terminated?
7. Why should destructive operations always be independently validated afterward?
