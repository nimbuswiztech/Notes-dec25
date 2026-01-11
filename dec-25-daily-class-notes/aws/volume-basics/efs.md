# efs

Amazon Elastic File System (EFS) is a scalable, cloud-native NFS file system for use with AWS Cloud services and on‑premises resources. This document provides a hands-on EFS training structure and a detailed Ubuntu 22.04 LTS demo guide suitable for workshops.

## EFS Practical Session — Structure

1. Introduction to Amazon EFS
   * Explain what EFS is, its features, and core use cases.
   * Compare EFS with other AWS storage solutions (EBS, S3).
2. Setting Up EFS: Demo overview
   * Prerequisites:
     * An AWS account with appropriate permissions
     * At least two Amazon EC2 instances in the same VPC
   * High-level demo steps:
     * Create a file system
     * Configure mount targets (one per AZ)
     * Install NFS utilities on EC2 instances
     * Mount the EFS file system
     * Test file sharing
     * (Optional) Basic performance testing
3. Best Practices and Security
   * Manage permissions with AWS IAM and POSIX.
   * Backup strategies (AWS Backup).
   * Network security (restrict access, encryption options).

Tips for running the session:

* Prepare resources (EC2 instances, IAM roles) in advance.
* Use diagrams to illustrate architecture and data flow.
* Let participants follow along and perform steps themselves.
* Pause for questions frequently and highlight troubleshooting tips.
* Provide a quick reference sheet for key CLI commands and console actions.

Scenario-based discussion table (use these to drive discussion):

| Scenario                                                                | Question                                                                          |
| ----------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| Multiple web servers must serve the same static content                 | How would you set up EFS so files are accessible and writable by all servers?     |
| A user reports high latency from one EC2 instance accessing EFS         | What troubleshooting steps would you follow?                                      |
| You need to restrict access to EFS only to certain applications/servers | Which security mechanisms (AWS and OS level) would you use to enforce this?       |
| Unexpectedly high storage costs are observed with EFS                   | What tools/features help monitor and control EFS storage costs?                   |
| A requirement mandates all file transfers be encrypted in transit       | Does EFS support encryption in transit, and how would you enable and verify it?   |
| A file system needs backup and recovery support                         | How can you integrate EFS with AWS native backup and disaster recovery solutions? |

***

## Amazon EFS Hands-On Demo Guide (Ubuntu)

This guide walks through a production-grade hands-on demo on Ubuntu 22.04 LTS EC2 instances. It covers networking and security, mounting (including TLS), automount, benchmarking, monitoring, troubleshooting, and cleanup.

{% hint style="info" %}
Time estimate: 40–60 minutes for the full workflow (including benchmark and cleanup).
{% endhint %}

### Overview / Goals

* Build a secure, multi‑AZ architecture where two Ubuntu servers share the same filesystem.
* Install and compare the EFS mount helper (`amazon-efs-utils`) and the native NFS client (`nfs-common`).
* Enable encryption in transit (TLS).
* Persist mounts across reboots via `/etc/fstab`.
* Run `fio` benchmarks.
* Collect CloudWatch metrics and set alerts.
* Troubleshoot common errors (e.g., `mount.nfs4: Connection timed out`).

### Architecture (logical)

* VPC (10.0.0.0/16) with two subnets in different AZs.
* EFS file system with one mount target per AZ.
* Security groups:
  * `sg-ec2` for EC2 instances (outbound TCP 2049 allowed).
  * `sg-efs` for mount targets (inbound TCP 2049 from `sg-ec2`).
* IAM role `EC2EFSRole` with `AmazonElasticFileSystemsUtils` for mount helper logs and optional CloudWatch publishing.

{% stepper %}
{% step %}
### Step 1 — Create Security Groups

* In the VPC/EC2 Console, create `sg-ec2`:
  * Inbound: SSH 22 from your IP.
  * Outbound: allow all (default).
* Create `sg-efs`:
  * Inbound: NFS TCP 2049 with Source = `sg-ec2` (use security group ID).
  * Outbound: default (all).

This ensures only instances in `sg-ec2` can reach EFS.
{% endstep %}

{% step %}
### Step 2 — Create the EFS File System

* Open EFS Console ➜ Create file system.
* Choose Quick create or Customize:
  * Performance mode: General Purpose.
  * Throughput mode: Elastic (default).
  * Encryption at rest: enabled by default.
  * Lifecycle management: default.
* Click Create. Status shows Creating then Available within \~60 seconds.

Note: Mount targets are auto-provisioned for selected subnets.
{% endstep %}

{% step %}
### Step 3 — Attach the EFS Security Group

* In EFS Console ➜ Network ➜ Manage.
* For each mount target, replace the default group with `sg-efs`.
* Wait until lifecycle state returns to Available.
{% endstep %}

{% step %}
### Step 4 — Launch Two Ubuntu 22.04 EC2 Instances

* EC2 Console ➜ Launch instance.
  * AMI: Ubuntu Server 22.04 LTS.
  * Instance type: t3.micro (demo scale).
  * Network/subnet:
    * EC2-A → Subnet-A (AZ 1).
    * EC2-B → Subnet-B (AZ 2).
  * IAM role: `EC2EFSRole`.
  * Security group: attach `sg-ec2`.
  * Storage: default 8 GiB gp3.
* Launch and note public IPs for SSH.
{% endstep %}

{% step %}
### Step 5 — Install Required Packages on Ubuntu

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

(amazon-efs-utils ≥ 2.3 supports IPv6 and improved TLS.)
{% endstep %}

{% step %}
### Step 6 — Mount EFS

Set the file system ID and mount point variables:

```bash
FILE_SYSTEM_ID=fs-XYZ123   # replace with your FS ID
MOUNT_POINT=/mnt/efs
sudo mkdir -p $MOUNT_POINT
```

#### 6-A. Mount with the EFS Mount Helper (Recommended)

```bash
sudo mount -t efs $FILE_SYSTEM_ID:/ $MOUNT_POINT
```

The helper applies optimal NFS options (rsize/wsize=1 MiB, hard, timeo=600, retrans=2).

#### 6-B. Mount via Native NFS4 Client (Fallback)

```bash
REGION=us-east-1
AZ=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone)
DNS="$AZ.$FILE_SYSTEM_ID.efs.$REGION.amazonaws.com"
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport $DNS:/ $MOUNT_POINT
```

Use this if `amazon-efs-utils` is not available.
{% endstep %}

{% step %}
### Step 7 — Verify Shared Storage

On EC2-A:

```bash
echo "Hello from EC2-A" | sudo tee /mnt/efs/hello.txt
```

On EC2-B:

```bash
cat /mnt/efs/hello.txt
# Expected output: Hello from EC2-A
```

Bidirectional visibility confirms the shared filesystem.
{% endstep %}

{% step %}
### Step 8 — Enable Encryption in Transit (TLS)

Using the mount helper to enable TLS:

```bash
sudo umount /mnt/efs
sudo mount -t efs -o tls $FILE_SYSTEM_ID:/ /mnt/efs
```

This uses a local stunnel proxy (ports 20049–21049) to encrypt NFS traffic. Audit logs at:

* /var/log/amazon/efs/mount.log
{% endstep %}

{% step %}
### Step 9 — Auto-Mount on Reboot

Add to /etc/fstab:

```bash
sudo bash -c 'echo "$FILE_SYSTEM_ID:/ /mnt/efs efs _netdev,tls 0 0" >> /etc/fstab'
```

Test:

```bash
sudo umount /mnt/efs
sudo mount -a
```

Reboot and verify with `df -h`.
{% endstep %}

{% step %}
### Step 10 — Optional Performance Benchmark

Install fio and run sequential benchmarks:

```bash
sudo apt-get install -y fio
fio --name=seq-read --directory=/mnt/efs --rw=read --bs=4M --size=4G
fio --name=seq-write --directory=/mnt/efs --rw=write --bs=4M --size=4G --end_fsync=1
```

Observe throughput differences when using:

* Elastic throughput (default),
* Provisioned throughput,
* Bursting throughput.
{% endstep %}

{% step %}
### Step 11 — Monitor with CloudWatch

EFS emits CloudWatch metrics; add to a dashboard.

Important metrics:

* BurstCreditBalance — remaining credits (Bursting mode).
* PercentIOLimit — I/O utilization (General Purpose).
* DataReadIOBytes, DataWriteIOBytes — throughput aggregates.

Typical alert thresholds:

* BurstCreditBalance < 20%
* PercentIOLimit > 80% for 15 minutes

Consider installing the CloudWatch agent on EC2 to capture per‑instance metrics.
{% endstep %}

{% step %}
### Step 12 — Cleanup

On each EC2 instance:

```bash
sudo umount /mnt/efs
```

* EFS Console ➜ File system ➜ Delete (mount targets removed automatically).
* Terminate EC2 instances.
* Remove security groups if unused.
{% endstep %}
{% endstepper %}

### Troubleshooting Checklist

| Symptom                                | Likely Cause                                    | Fix                                                           |
| -------------------------------------- | ----------------------------------------------- | ------------------------------------------------------------- |
| `mount.nfs4: Connection timed out`     | Missing NFS 2049 in either security group       | Reverify rules: inbound on `sg-efs`, outbound on `sg-ec2`     |
| `aws: error: EFS mount helper missing` | `amazon-efs-utils` not installed                | `sudo apt-get install -y amazon-efs-utils`                    |
| `Access denied by server`              | POSIX UID/GID mismatch when using Access Points | Align UID/GID or use EFS Access Point with appropriate config |
| Slow writes, high latency              | Exceeded PercentIOLimit (General Purpose)       | Switch to Elastic/Provisioned throughput or Max I/O           |
| Reboot loses mount                     | fstab entry missing `_netdev` or wrong FS type  | Use `efs` type with `_netdev,tls`                             |

***

## Scenario-Based Quiz (expand to view answers)

<details>

<summary>New compliance rule mandates TLS for all data in transit — Which single mount option enforces this and how do you verify it?</summary>

Answer:

* Mount option: `-o tls` (with the EFS mount helper or in /etc/fstab as `efs ... _netdev,tls`).
* Verify: check that the mount is using the EFS mount helper (mount output) and inspect logs at `/var/log/amazon/efs/mount.log`; network traffic is proxied through local stunnel (ports 20049–21049).

</details>

<details>

<summary>Your workload spikes to 250 MiB/s for 10 minutes every hour — Which throughput mode minimizes cost without throttling?</summary>

Answer:

* Consider Provisioned throughput if sustained throughput is required and predictable.
* If bursts are short and your stored data size accrues enough baseline throughput, Elastic or Bursting may suffice. Choose based on cost vs predictable sustained throughput needs.

</details>

<details>

<summary>Server team reports intermittent `stale file handle` errors — List investigation steps specific to EFS NFSv4.1 semantics.</summary>

Answer:

* Check network stability between the client and mount target.
* Verify mount options and NFS version; remount with correct options.
* Examine server-side lifecycle events (file system or mount target changes).
* Check for client-side caching, stale mounts, or re-created inodes (e.g., deleted and re-created files).

</details>

<details>

<summary>You must restrict write access to a single application user while letting others read — Which combination of POSIX permissions, EFS Access Point, and IAM would you choose and why?</summary>

Answer:

* Use an EFS Access Point configured with the required POSIX UID/GID and root directory permissions, ensuring the application connects via that access point.
* POSIX permissions on files/directories restrict write access.
* IAM can control who can create/delete access points or manage EFS resources, but POSIX+Access Point enforce runtime access.

</details>

<details>

<summary>After moving to Max I/O mode, latency doubled — Explain why and propose remediation.</summary>

Answer:

* Max I/O optimizes throughput and metadata scaling but can increase latency due to higher parallelism and distributed metadata.
* Remediation: revert to General Purpose for latency-sensitive workloads, or tune the application for higher concurrency and avoid small synchronous writes.

</details>

***

## Appendix — Latency & Storage Class Table

| Storage Class         | Typical Read Latency | Typical Write Latency | Use Case Example             |
| --------------------- | -------------------- | --------------------- | ---------------------------- |
| EFS Standard          | 250 µs               | 2.7 ms                | CMS, CI/CD, home directories |
| EFS Infrequent Access | \~10–50 ms           | \~10–50 ms            | Archive, disaster recovery   |
| EFS Archive           | \~tens of ms         | \~tens of ms          | Regulatory cold storage      |

***

If you want, I can:

* Produce a shorter one-page cheat sheet of key CLI commands for the workshop.
* Convert the stepper into separate GitBook pages per step.
* Generate an /etc/fstab example file with multiple options for different scenarios.
