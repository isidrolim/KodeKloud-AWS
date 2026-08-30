# Change EC2 Instance Type via AWS CLI

## Scenario

The Nautilus DevOps team identified an underutilized EC2 instance and decided to reduce its instance size to optimize resource usage.

The existing `devops-ec2` instance was running as `t2.micro` and needed to be changed to `t2.nano`.

## Requirement

- **Instance Name:** `devops-ec2`
- **Original Type:** `t2.micro`
- **New Type:** `t2.nano`
- **Region:** `us-east-1`
- **Required Final State:** `running`

## Initial State

Verify the AWS CLI environment.

```bash
aws sts get-caller-identity
aws configure list
aws configure get region
```

Locate the target instance and inspect its current configuration.

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=devops-ec2" \
  --query "Reservations[*].Instances[*].[InstanceId,State.Name,InstanceType]" \
  --output table \
  --region us-east-1
```

The instance was confirmed as:

```text
Name:          devops-ec2
Instance Type: t2.micro
State:         running
```

## Troubleshooting Path

Changing the EC2 instance type requires the instance to be stopped first.

```text
Locate devops-ec2
        ↓
Verify current type = t2.micro
        ↓
Stop instance
        ↓
Wait until stopped
        ↓
Modify instance type
        ↓
Verify type = t2.nano
        ↓
Start instance
        ↓
Wait until running
        ↓
Final validation
```

## Verification Before Fix

Identify the exact EC2 instance ID before performing any modification.

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=devops-ec2" \
  --query "Reservations[*].Instances[*].[InstanceId,State.Name,InstanceType]" \
  --output table \
  --region us-east-1
```

Target instance:

```text
i-05119c131f5a7a184
```

## Systematic Elimination

The instance cannot safely have its instance type changed while running.

Stop it first:

```bash
aws ec2 stop-instances \
  --instance-ids i-05119c131f5a7a184 \
  --region us-east-1
```

Wait until AWS confirms that the instance is fully stopped:

```bash
aws ec2 wait instance-stopped \
  --instance-ids i-05119c131f5a7a184 \
  --region us-east-1
```

Verify:

```bash
aws ec2 describe-instances \
  --instance-ids i-05119c131f5a7a184 \
  --query "Reservations[0].Instances[0].[InstanceId,State.Name,InstanceType]" \
  --output table \
  --region us-east-1
```

Expected intermediate state:

```text
i-05119c131f5a7a184
stopped
t2.micro
```

## First Finding

The instance was successfully stopped, allowing the instance type to be modified.

AWS CLI help was used to discover the correct syntax:

```bash
aws ec2 modify-instance-attribute help | less
```

Relevant parameters:

```text
--instance-id
--instance-type
```

## Fix

Change the instance type from `t2.micro` to `t2.nano`.

```bash
aws ec2 modify-instance-attribute \
  --instance-id i-05119c131f5a7a184 \
  --instance-type t2.nano \
  --region us-east-1
```

Before starting the instance, verify that the modification actually succeeded:

```bash
aws ec2 describe-instances \
  --instance-ids i-05119c131f5a7a184 \
  --query "Reservations[0].Instances[0].[InstanceId,State.Name,InstanceType]" \
  --output table \
  --region us-east-1
```

Expected intermediate result:

```text
i-05119c131f5a7a184
stopped
t2.nano
```

Start the instance:

```bash
aws ec2 start-instances \
  --instance-ids i-05119c131f5a7a184 \
  --region us-east-1
```

Wait until it reaches the required running state:

```bash
aws ec2 wait instance-running \
  --instance-ids i-05119c131f5a7a184 \
  --region us-east-1
```

## Validation

Validate against the original requirement:

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=devops-ec2" \
  --query "Reservations[*].Instances[*].[InstanceId,State.Name,InstanceType]" \
  --output table \
  --region us-east-1
```

Final result:

```text
Instance: devops-ec2
State:    running
Type:     t2.nano
```

The original success criteria were satisfied:

```text
t2.micro
    ↓
STOP
    ↓
t2.nano
    ↓
START
    ↓
running
```

## Lessons Learned

- An EBS-backed EC2 instance normally needs to be stopped before changing its instance type.
- `stop-instances` initiates the shutdown but does not mean the instance is immediately stopped.
- `aws ec2 wait instance-stopped` provides a cleaner way to wait for the required state.
- `modify-instance-attribute` changes properties of an existing EC2 instance.
- `--instance-id` identifies which instance AWS should modify.
- `--instance-type` specifies the new instance type.
- Verify the new instance type **before starting the instance again**.
- `aws ec2 wait instance-running` can be used after restarting.
- Final validation should confirm both the configuration and operational state.

## Engineering Insight

Do not combine several infrastructure changes and only inspect the result at the end.

Use checkpoints:

```text
BEFORE
devops-ec2
running
t2.micro
        ↓
STOP
        ↓
CHECKPOINT 1
stopped
t2.micro
        ↓
MODIFY
        ↓
CHECKPOINT 2
stopped
t2.nano
        ↓
START
        ↓
CHECKPOINT 3
running
t2.nano
```

This makes troubleshooting much easier because if something fails, we know exactly which transition introduced the problem.

For production systems, resizing also requires considering:

```text
Application downtime
        +
Instance-type compatibility
        +
CPU / memory requirements
        +
Network performance
        +
EBS optimization
        +
Architecture compatibility
        +
Maintenance window
```

## Knowledge Check

1. Why did we stop the EC2 instance before changing its instance type?
2. What is the difference between `stop-instances` and `wait instance-stopped`?
3. Why did we verify `t2.nano` while the instance was still stopped?
4. Which command changes the instance type?
5. Why does `modify-instance-attribute` require both `--instance-id` and `--instance-type`?
6. What two properties prove that the original requirement was completely satisfied?
7. Why are intermediate checkpoints safer than making several changes and validating only at the end?
