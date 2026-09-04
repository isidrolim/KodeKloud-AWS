# AWS Level 2 – Task 006: Setting Up an EC2 Instance and CloudWatch Alarm

## Scenario
The Nautilus DevOps team needs an EC2 instance for an application and a CloudWatch alarm to monitor CPU utilization.

The alarm must trigger when CPU utilization reaches or exceeds **90% for one consecutive 5-minute period** and send a notification to the existing SNS topic `nautilus-sns-topic`.

## Requirements

- **Region:** `us-east-1`
- **EC2 Name:** `nautilus-ec2`
- **AMI:** Appropriate Ubuntu AMI
- **CloudWatch Alarm:** `nautilus-alarm`
- **Metric:** CPUUtilization
- **Statistic:** Average
- **Threshold:** >= 90%
- **Period:** 5 minutes
- **Evaluation Period:** 1
- **SNS Topic:** `nautilus-sns-topic`

## Steps

### 1. Launch the EC2 Instance

Go to:

`EC2 → Instances → Launch instances`

Configure:

- **Name:** `nautilus-ec2`
- **AMI:** Ubuntu
- Select an appropriate instance type
- Keep the remaining settings appropriate for the lab

Click **Launch instance**.

Wait until the instance reaches:

`Running`

---

### 2. Locate the Existing SNS Topic

Go to:

`Amazon SNS → Topics`

Confirm the following topic already exists:

`nautilus-sns-topic`

No new SNS topic needs to be created.

---

### 3. Create the CloudWatch Alarm

Go to:

`CloudWatch → Alarms → All alarms → Create alarm`

Click:

`Select metric`

Navigate to:

`EC2 → Per-Instance Metrics`

Find the `CPUUtilization` metric for the newly created `nautilus-ec2` instance.

Select it and click:

`Select metric`

---

### 4. Configure the Metric

Configure:

- **Statistic:** Average
- **Period:** 5 minutes

Under **Conditions**:

- **Threshold type:** Static
- **Whenever CPUUtilization is:** Greater/Equal (>=)
- **Threshold value:** `90`

Configure:

- **Datapoints to alarm:** `1 out of 1`

This means:

`Average CPU >= 90% for 1 consecutive 5-minute period → ALARM`

---

### 5. Configure the Alarm Action

Under **Notification**:

- **Alarm state trigger:** In alarm
- **SNS topic:** Select existing SNS topic
- **Topic:** `nautilus-sns-topic`

Continue to the next step.

---

### 6. Name the Alarm

Set:

`Alarm name: nautilus-alarm`

Review the configuration and click:

`Create alarm`

---

## Validation

Verify the EC2 instance:

`EC2 → Instances → nautilus-ec2`

Confirm:

- Instance exists
- Instance is `Running`
- Ubuntu AMI is being used

Then go to:

`CloudWatch → Alarms → All alarms → nautilus-alarm`

Confirm:

- **Metric:** CPUUtilization
- **Statistic:** Average
- **Period:** 5 minutes
- **Threshold:** >= 90%
- **Datapoints to alarm:** 1 of 1
- **Notification:** `nautilus-sns-topic`

The alarm may initially show `Insufficient data` until CloudWatch receives enough metric data.

## Result

The monitoring path is:

`nautilus-ec2 → CPUUtilization → CloudWatch → nautilus-alarm → nautilus-sns-topic`

The EC2 instance is now monitored for high CPU utilization, with the alarm configured to trigger when average CPU utilization reaches or exceeds 90% for one consecutive 5-minute period.

**Task Status:** ✅ Completed
