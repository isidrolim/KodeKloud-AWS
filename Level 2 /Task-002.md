# Expand EC2 Root EBS Volume and Filesystem

## Scenario

The Nautilus DevOps team reported that the development EC2 instance `xfusion-ec2` was running out of storage.

The existing root EBS volume was `8 GiB` and needed to be expanded to `12 GiB` without disrupting the running instance. The additional capacity also needed to be reflected inside the Linux operating system.

## Requirement

- **Instance:** `xfusion-ec2`
- **Original root volume:** `8 GiB`
- **Required root volume:** `12 GiB`
- **Root mount:** `/`
- **Region:** `us-east-1`
- **SSH key:** `/root/xfusion-keypair.pem`
- Expand the AWS EBS volume.
- Ensure the Linux root partition/filesystem reflects the expanded capacity.

## Initial State

Build the dependency path before changing storage:

```text
EC2 Instance
     ↓
Root EBS Volume
     ↓
Expand EBS: 8 GiB → 12 GiB
     ↓
Linux block device
     ↓
Root partition
     ↓
Root filesystem /
     ↓
Validate usable capacity
```

Identify the target EC2 instance:

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=xfusion-ec2" \
  --query "Reservations[0].Instances[0].[InstanceId,ImageId,PrivateIpAddress,PublicIpAddress,State.Name]" \
  --output table \
  --region us-east-1
```

The instance was identified as:

```text
Instance ID: i-0a09160cbc41da06b
AMI:         ami-0c101f26f147fa7fd
State:       running
```

## Troubleshooting Path

The EBS volume was expanded from the AWS side first.

The next challenge was accessing the Linux instance to verify whether the guest operating system also recognized the additional capacity.

Initial SSH attempt using the private IP did not establish a connection:

```bash
ssh -i /root/xfusion-keypair.pem \
  ec2-user@172.31.82.63
```

This demonstrated an important distinction:

```text
AWS resource exists
        ≠
Private IP is necessarily reachable
from the current administration host
```

The instance's public IP was used instead.

## Verification Before Fix

Determine the operating system from the AMI rather than guessing the SSH username:

```bash
aws ec2 describe-images \
  --image-ids ami-0c101f26f147fa7fd \
  --query "Images[0].[Name,Description,PlatformDetails,Architecture]" \
  --output table \
  --region us-east-1
```

Result:

```text
Amazon Linux 2023
Linux/UNIX
x86_64
```

Therefore the appropriate default SSH user was:

```text
ec2-user
```

Protect the provided SSH private key:

```bash
chmod 400 /root/xfusion-keypair.pem
```

Connect using the reachable public IP:

```bash
ssh -i /root/xfusion-keypair.pem \
  ec2-user@<PUBLIC-IP>
```

## Systematic Elimination

Once connected, inspect the storage stack before running any resize command:

```bash
lsblk
```

The result showed:

```text
NAME       SIZE TYPE MOUNTPOINTS
xvda        12G disk
└─xvda1     12G part /
```

Additional boot partitions were also present:

```text
xvda127      1M part
xvda128     10M part /boot/efi
```

This was the critical finding.

## First Finding

Both storage layers already reflected the new capacity:

```text
AWS EBS
8 GiB
   ↓
Expanded
   ↓
12 GiB

Linux
/dev/xvda
12 GiB
   ↓
/dev/xvda1
12 GiB
   ↓
/
```

Therefore, manually running a partition expansion command such as:

```bash
growpart /dev/xvda 1
```

was unnecessary.

The key engineering principle was:

> Inspect the current state before running a resize command.

## Fix

The AWS EBS root volume was expanded from:

```text
8 GiB
```

to:

```text
12 GiB
```

After connecting to the Amazon Linux 2023 instance, `lsblk` confirmed that the root partition had already expanded to the full `12 GiB`.

No additional partition modification was required.

If the partition had remained at approximately `8 GiB`, the next step would have been to determine the partition layout and filesystem type before selecting the appropriate expansion command.

For example:

```bash
lsblk
df -Th /
```

Only after that evidence would commands such as `growpart`, `xfs_growfs`, or `resize2fs` be considered.

## Validation

Verify the block-device layout:

```bash
lsblk
```

Confirmed:

```text
xvda     12G disk
xvda1    12G part /
```

For additional filesystem-level validation:

```bash
df -Th /
```

The success path was therefore:

```text
EBS volume = 12 GiB       ✓
        ↓
Linux disk = 12 GiB       ✓
        ↓
Root partition = 12 GiB   ✓
        ↓
Root mounted on /         ✓
```

The KodeKloud challenge validation completed successfully.

## Lessons Learned

- Expanding an EBS volume and expanding a filesystem are separate layers.
- Always identify the correct EC2 instance and root volume before modifying storage.
- EBS volumes can normally be expanded while an instance remains running.
- After expanding EBS, inspect the guest operating system with `lsblk`.
- Do not automatically run `growpart` after every EBS expansion.
- First determine whether the partition has already consumed the additional space.
- `df -Th /` identifies both filesystem capacity and filesystem type.
- Filesystem type matters because XFS and ext4 use different expansion tools.
- Amazon Linux normally uses the `ec2-user` SSH account.
- A private EC2 address may not be reachable from the administration host even when the instance itself is healthy.
- Use the reachable network path appropriate for the environment.

## Engineering Insight

Storage expansion should be understood as a layered operation:

```text
AWS EBS Volume
      │
      │ modify volume
      ▼
Linux Block Device
      │
      │ partition expansion if required
      ▼
Partition
      │
      │ filesystem expansion if required
      ▼
Filesystem
      │
      ▼
Application usable space
```

A common mistake is seeing:

```text
EBS = 12 GiB
```

and immediately assuming:

```text
growpart
xfs_growfs
```

Instead:

```text
CHANGE
   ↓
OBSERVE
   ↓
COMPARE TO REQUIREMENT
   ↓
CHANGE NEXT LAYER ONLY IF NECESSARY
```

In this task, observation showed:

```text
/dev/xvda  = 12G
/dev/xvda1 = 12G
```

so changing the partition again would have provided no benefit.

## Knowledge Check

1. What is the difference between an EBS volume, partition, and filesystem?
2. Why should `lsblk` be run after expanding an EBS volume?
3. What would it mean if `/dev/xvda` showed `12G` but `/dev/xvda1` still showed `8G`?
4. Why should `df -Th /` be checked before choosing a filesystem expansion command?
5. Which SSH username is normally used for Amazon Linux?
6. Why did the private IP not necessarily provide a usable SSH path from `aws-client`?
7. Why was `growpart` unnecessary in this particular task?
8. What is the safest general workflow for online storage expansion?

```text
Identify → Expand → Inspect → Extend if required → Validate
```
