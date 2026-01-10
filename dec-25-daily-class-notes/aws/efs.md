# efs

## EFS

### EFS Practical Session

Amazon Elastic File System (EFS) is a scalable, cloud-native NFS file system for use with AWS Cloud services and on‑premises resources. Here’s how you can organize and deliver an engaging, hands‑on EFS training session, including a structured demo and scenario‑based questions for participants.

#### Session Structure

{% stepper %}
{% step %}
### Introduction to Amazon EFS

* Explain what EFS is, its features, and core use cases.
* Compare EFS with other AWS storage solutions (EBS, S3).
{% endstep %}

{% step %}
### Setting Up EFS: Step-by-Step Demo (overview)

Prerequisites

* An AWS account with appropriate permissions
* At least two Amazon EC2 instances in the same VPC

Steps

* Create a File System (EFS Console -> Create file system)
* Configure mount targets (ensure each AZ has a mount target; allow NFS traffic TCP 2049 in SGs)
* Install NFS utilities on EC2 instances (e.g., `sudo yum install -y nfs-utils` or `sudo apt-get install nfs-common`)
*   Mount the EFS file system:

    ```
    sudo mount -t nfs4 -o nfsvers=4.1 <EFS-DNS>:/ /mnt/efs
    ```
* Test file sharing: create files on Instance A and verify from Instance B
* Optional: basic performance testing (copy files of various sizes)
{% endstep %}

{% step %}
### Best Practices and Security

* Review managing permissions with AWS IAM and POSIX.
* Discuss backup strategies (AWS Backup).
* Network security: restricting access, encryption options.
{% endstep %}
{% endstepper %}

#### Tips for Running a Great Practical Session

* Prepare all resources in advance, including EC2 instances and necessary IAM roles.
* Use a whiteboard or diagrams to illustrate EFS architecture and data flow.
* Allow participants to follow along and perform each step themselves.
* Pause frequently for questions and clarify each step.
* If possible, simulate a “real” scenario (e.g., sharing media files between web servers).
* Highlight troubleshooting tips: e.g., NFS mount failures, permission errors.
* Provide a quick reference sheet for key CLI commands and console actions.

#### Scenario-Based Questions for Discussion

| Scenario                                                                | Question                                                                          |
| ----------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| Multiple web servers must serve the same static content                 | How would you set up EFS so files are accessible and writable by all servers?     |
| A user reports high latency from one EC2 instance accessing EFS         | What troubleshooting steps would you follow?                                      |
| You need to restrict access to EFS only to certain applications/servers | Which security mechanisms (AWS and OS level) would you use to enforce this?       |
| Unexpectedly high storage costs are observed with EFS                   | What tools/features help monitor and control EFS storage costs?                   |
| A requirement mandates all file transfers be encrypted in transit       | Does EFS support encryption in transit, and how would you enable and verify it?   |
| A file system needs backup and recovery support                         | How can you integrate EFS with AWS native backup and disaster recovery solutions? |

By following this structure and encouraging hands-on participation, your EFS practical session will be clear, engaging, and impactful. Use the scenario questions to drive discussion and ensure deeper understanding.

***

## Amazon EFS Hands-On Demo Guide (Ubuntu)

Amazon Elastic File System (EFS) offers fully managed, elastic, NFS-compatible shared storage for AWS workloads. This in-depth guide walks you through a complete, production-grade hands-on demonstration on Ubuntu 22.04 LTS EC2 instances. It covers networking and security, mounting with TLS, automating re-mounts on reboot, benchmarking, monitoring, troubleshooting, and cleanup. Scenario-based quiz questions at the end help reinforce each concept.

### Overview

Running a live demo is the fastest way to learn how EFS behaves. In the following workflow you will:

* Build a secure, multi-AZ architecture that lets two Ubuntu servers share the same filesystem.
* Install and compare the EFS mount helper (`amazon-efs-utils`) and the native NFS client (`nfs-common`).
* Enable encryption in transit with one command.
* Persist mounts across reboots using `/etc/fstab`.
* Run a simple `fio` benchmark to observe I/O patterns under different throughput modes.
* Collect CloudWatch metrics and set alerts on throughput saturation.
* Resolve common errors such as `mount.nfs4: Connection timed out`.

Time estimate: 40–60 minutes for the full workflow, including benchmark and cleanup.

### Demo Prerequisites

| Component   | Minimum Requirement                                             | Why It’s Needed                      |
| ----------- | --------------------------------------------------------------- | ------------------------------------ |
| AWS account | IAM user/role with permission to create EFS, EC2 and CloudWatch | Resource provisioning                |
| Region      | Any region that supports EFS (e.g., `us-east-1`)                | Service availability                 |
| VPC         | Default VPC or custom VPC with at least two public subnets      | Multi-AZ demo topology               |
| Ubuntu AMI  | 22.04 LTS or newer                                              | Package names match tutorial         |
| Local tools | SSH client, AWS CLI v2                                          | Instance access & optional CLI steps |

Environment Architecture (logical)

* VPC (10.0.0.0/16) with Subnet-A (us-east-1a) for EC2-A and Subnet-B (us-east-1b) for EC2-B.
* EFS file system (e.g., `fs-XYZ123`) with one mount target per AZ.
* Security groups: `sg-ec2` for instances (outbound TCP 2049 allowed), `sg-efs` for mount targets (inbound TCP 2049 from `sg-ec2`).
* IAM role `EC2EFSRole` with `AmazonElasticFileSystemsUtils`.

### Step-by-Step Hands-On

{% stepper %}
{% step %}
#### Step 1 — Create Security Groups

1. Launch VPC Console ➜ Security Groups ➜ Create security group
   * Name: `sg-ec2`
   * Inbound: SSH 22 from your IP.
   * Outbound: allow all traffic (default).
2. Create second group
   * Name: `sg-efs`
   * Inbound: NFS TCP 2049 with Source = `sg-ec2` (use the security group ID dropdown).
   * Outbound: default (all traffic).

This least-privilege design means only instances in `sg-ec2` can reach EFS.
{% endstep %}

{% step %}
#### Step 2 — Create the EFS File System

1. Open EFS Console ➜ Create file system.
2. Choose Quick create or Customize:
   * Performance mode: General Purpose.
   * Throughput mode: Elastic.
   * Encryption at rest: enabled by default.
   * Lifecycle management: keep default.
3. Click Create. Status shows Creating then Available within \~60 seconds.

Mount targets are auto-provisioned—each picks an IP from the subnet you selected.
{% endstep %}

{% step %}
#### Step 3 — Attach the EFS Security Group

1. In the EFS console: Network ➜ Manage.
2. For every mount target, replace the default group with `sg-efs`.
3. Wait until the life-cycle state returns to Available.
{% endstep %}

{% step %}
#### Step 4 — Launch Two Ubuntu 22.04 EC2 Instances

1. EC2 Console ➜ Launch instance.
2. AMI: Ubuntu Server 22.04 LTS.
3. Instance type: t3.micro (demo scale).
4. Network & subnet:
   * EC2-A -> Subnet-A (us-east-1a).
   * EC2-B -> Subnet-B (us-east-1b).
5. IAM role: `EC2EFSRole`.
6. Security group: attach `sg-ec2`.
7. Storage: default 8 GiB gp3.
8. Launch and note both public IPs for SSH.
{% endstep %}

{% step %}
#### Step 5 — Install Required Packages on Ubuntu

SSH into EC2-A (repeat on EC2-B):

```bash
sudo apt-get update -y               # refresh repos
sudo apt-get install -y nfs-common   # native NFS client
sudo apt-get install -y amazon-efs-utils  # EFS mount helper
```

Verify version:

```bash
dpkg -l | grep amazon-efs-utils
```

`amazon-efs-utils` ≥ 2.3 supports IPv6 and improved TLS.
{% endstep %}

{% step %}
#### Step 6 — Mount EFS

6-A. Mount with the EFS Mount Helper (Recommended)

```bash
FILE_SYSTEM_ID=fs-XYZ123          # replace
MOUNT_POINT=/mnt/efs
sudo mkdir -p $MOUNT_POINT
sudo mount -t efs $FILE_SYSTEM_ID:/ $MOUNT_POINT
```

The helper auto-injects optimal NFS options (rsize/wsize=1 MiB, hard, timeo=600, retrans=2).

6-B. Mount via Native NFS4 Client (Fallback)

```bash
REGION=us-east-1
AZ=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone)
DNS="$AZ.$FILE_SYSTEM_ID.efs.$REGION.amazonaws.com"
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport $DNS:/ $MOUNT_POINT
```

Use this when the mount helper package is unavailable.
{% endstep %}

{% step %}
#### Step 7 — Verify Shared Storage

On EC2-A:

```bash
echo "Hello from EC2-A" | sudo tee /mnt/efs/hello.txt
```

On EC2-B:

```bash
cat /mnt/efs/hello.txt
# Output: Hello from EC2-A
```

Bidirectional visibility confirms the file system is shared.
{% endstep %}

{% step %}
#### Step 8 — Enable Encryption in Transit (TLS)

Mount helper makes TLS frictionless:

```bash
sudo umount /mnt/efs
sudo mount -t efs -o tls $FILE_SYSTEM_ID:/ /mnt/efs
```

A local stunnel proxy (ports 20049–21049) encrypts NFS packets with TLS 1.2/AES-256. Audit log at: `/var/log/amazon/efs/mount.log`.
{% endstep %}

{% step %}
#### Step 9 — Auto-Mount on Reboot

Edit `/etc/fstab`:

```bash
sudo bash -c 'echo "$FILE_SYSTEM_ID:/ /mnt/efs efs _netdev,tls 0 0" >> /etc/fstab'
```

Test:

```bash
sudo umount /mnt/efs
sudo mount -a     # re-reads fstab
```

Reboot the instance and run `df -h` to confirm the mount persists.
{% endstep %}

{% step %}
#### Step 10 — Optional Performance Benchmark

Install and run `fio`:

```bash
sudo apt-get install -y fio
fio --name=seq-read --directory=/mnt/efs --rw=read --bs=4M --size=4G
fio --name=seq-write --directory=/mnt/efs --rw=write --bs=4M --size=4G --end_fsync=1
```

Observe throughput variations across:

* Elastic throughput (default)
* Provisioned throughput
* Bursting throughput (credits accrue at 50 KiB/s per GiB stored)
{% endstep %}

{% step %}
#### Step 11 — Monitor with CloudWatch

EFS emits real-time metrics—add them to a dashboard:

| Metric             | Description                             | Typical Alert Threshold  |
| ------------------ | --------------------------------------- | ------------------------ |
| BurstCreditBalance | Remaining credits (Bursting mode)       | < 20% credits            |
| PercentIOLimit     | I/O utilization in General Purpose mode | > 80% for 15 min         |
| DataReadIOBytes    | Aggregate read throughput               | Abnormal spike detection |
| DataWriteIOBytes   | Aggregate write throughput              | Abnormal spike detection |

You can also install the CloudWatch agent on EC2 to capture per-instance throughput.
{% endstep %}

{% step %}
#### Step 12 — Cleanup

On each EC2 instance:

```bash
sudo umount /mnt/efs
```

* EFS Console ➜ File system ➜ Delete (deletes mount targets automatically)
* Terminate EC2 instances
* Remove the security groups if unused
{% endstep %}
{% endstepper %}

### Troubleshooting Checklist

<details>

<summary>mount.nfs4: Connection timed out</summary>

Likely cause: Missing NFS 2049 in either security group.\
Fix: Reverify rules: inbound on `sg-efs`, outbound on `sg-ec2`.

</details>

<details>

<summary>aws: error: EFS mount helper missing</summary>

Likely cause: `amazon-efs-utils` not installed.\
Fix: `sudo apt-get install -y amazon-efs-utils`

</details>

<details>

<summary>Access denied by server</summary>

Likely cause: POSIX UID/GID mismatch when using Access Points.\
Fix: Align UID/GID or use EFS Access Point with root-squash disabled.

</details>

<details>

<summary>Slow writes, high latency</summary>

Likely cause: Exceeded `PercentIOLimit` in General Purpose mode.\
Fix: Switch to Elastic/Provisioned throughput or Max I/O.

</details>

<details>

<summary>Reboot loses mount</summary>

Likely cause: Fstab entry missing `_netdev` or wrong FS type.\
Fix: Use `efs` type with `_netdev,tls`.

</details>

### Scenario-Based Quiz (for workshops)

<details>

<summary>New compliance rule mandates TLS for all data in transit</summary>

Question: Which single mount option enforces this and how do you verify it?\
(Answer guidance: use `-o tls` with the EFS mount helper and verify by checking the mount helper logs and/or that stunnel is running; verify NFS traffic over TLS via logs at `/var/log/amazon/efs/mount.log`.)

</details>

<details>

<summary>Your workload spikes to 250 MiB/s for 10 minutes every hour</summary>

Question: Which throughput mode minimizes cost without throttling?\
(Answer guidance: consider Provisioned throughput if sustained high throughput is needed, or Elastic if bursts can be absorbed—choose based on cost and pattern.)

</details>

<details>

<summary>Server team reports intermittent `stale file handle` errors</summary>

Question: List at least three investigation steps specific to EFS NFSv4.1 semantics.\
(Answer guidance: check for client-side caching issues, confirm network stability between client and mount target, review NFS version and mount options, check for file system re-creation or accidental deletion.)

</details>

<details>

<summary>Restrict write access to a single application user while letting others read</summary>

Question: Which combination of POSIX permissions, EFS Access Point, and IAM would you choose and why?\
(Answer guidance: use an Access Point with an enforced UID/GID for the application, set POSIX permissions to allow write only for that UID/GID, and use IAM for administrative controls.)

</details>

<details>

<summary>After moving to Max I/O mode, latency doubled</summary>

Question: Explain why this is expected and propose remediation.\
(Answer guidance: Max I/O optimizes throughput at the expense of per-operation latency; consider reverting to General Purpose if latency-sensitive, or adjust application behavior.)

</details>

### Appendix — Latency & Storage Class Table

| Storage Class         | Typical Read Latency | Typical Write Latency | Use Case Example             |
| --------------------- | -------------------- | --------------------- | ---------------------------- |
| EFS Standard          | 250 µs               | 2.7 ms                | CMS, CI/CD, home directories |
| EFS Infrequent Access | \~10–50 ms           | \~10–50 ms            | Archive, disaster recovery   |
| EFS Archive           | \~tens of ms         | \~tens of ms          | Regulatory cold storage      |

***

If you want, I can:

* produce a shortened, slide-ready version of this demo;
* extract only the commands into a single runnable script; or
* convert the Scenario-Based Quiz into an interactive workshop worksheet. Which would you like?
