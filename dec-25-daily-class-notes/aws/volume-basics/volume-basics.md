# volume basics

We need **EBS (Elastic Block Store) volumes in AWS** because they provide **durable, persistent, block-level storage** that can be attached to EC2 instances, functioning like a hard drive for storing data that requires frequent updates or continuous access. EBS volumes are essential for operating systems, databases, application code, logs, and throughput‑intensive or I/O‑sensitive workloads. They persist independently from the life of the instance, so data remains intact even if the instance stops or terminates.

Volumes in AWS, specifically **Amazon EBS (Elastic Block Store) volumes**, are durable, block-level storage devices that you attach to Amazon EC2 instances to provide persistent storage, similar to a physical hard drive attached to a computer.

{% hint style="info" %}
EBS volumes provide persistent, block-level storage for EC2 instances and are typically used for boot volumes, databases, application code, logs, and other workloads requiring frequent/random I/O and durability independent of instance lifecycle.
{% endhint %}

## Key aspects of AWS EBS volumes

* **Block-level storage:** Volumes appear to the EC2 instance like a hard drive, allowing you to store files, install applications, or host databases.
* **Persistence:** Data on an EBS volume persists independently of the EC2 instance’s lifecycle. Data remains intact even if the instance is stopped or terminated.
* **Flexibility and scalability:** You can dynamically modify size, performance (IOPS), and volume type on live volumes without downtime, making them adaptable to changing workload requirements.
* **Multiple volume types:** AWS offers a variety of EBS volume types optimized for different use cases:
  * SSD-backed volumes: General Purpose SSD (gp2/gp3), Provisioned IOPS SSD (io1/io2) — for transactional workloads requiring low latency and high IOPS.
  * HDD-backed volumes: Throughput Optimized HDD (st1), Cold HDD (sc1) — for throughput‑intensive or infrequently accessed workloads.
* **High availability and durability:** Data on EBS volumes is automatically replicated within the same Availability Zone to protect against hardware failures and ensure durability, with some volume types providing up to 99.999% durability.
* **Backup and recovery:** You can create snapshots of EBS volumes, which are incremental backups stored in Amazon S3, enabling easy restoration or migration of data.
* **Security:** EBS volumes support encryption using AWS Key Management Service (KMS), protecting data at rest and in transit between instances and volumes.

## Key reasons to use EBS volumes in AWS

* **Persistent storage:** Unlike instance store volumes that are ephemeral and lost when an instance stops or terminates, EBS volumes retain data independently, enabling reliable long-term storage for EC2 instances.
* **Flexibility and scalability:** Dynamically resize volumes, adjust performance (such as IOPS), and change volume types on live workloads to support evolving application needs — from boot volumes to large databases.
* **Performance tailoring:** Choose among multiple volume types (General Purpose SSD, Provisioned IOPS SSD, Throughput Optimized HDD, Cold HDD) to optimize for cost, throughput, or IOPS depending on workload.
* **Data availability and durability:** Automatic replication within an Availability Zone prevents data loss from hardware failure, ensuring high availability and durability.
* **Multiple attachments and shared storage:** Some volume types support Multi-Attach, allowing a single volume to be attached to multiple Nitro-based instances for shared data access.
* **Use cases:** EBS volumes are suitable for operating system files, application code, databases, logs, temporary files, and throughput-intensive workloads that require persistent, high-performance storage.

<details>

<summary>EBS vs other AWS storage options (S3, RDS)</summary>

* EBS provides block-level, persistent storage that behaves like a traditional disk attached to an EC2 instance — suitable for OS boot drives and workloads requiring frequent random I/O.
* S3 provides object storage designed for durability and scale (not a block device) — ideal for backups, static assets, and large objects.
* RDS provides managed databases (including underlying storage abstractions) but does not expose raw block devices for direct use by EC2 instances.

In short: EBS uniquely provides block-level, persistent disk storage attached to EC2 instances, distinct from object storage (S3) and managed database services (RDS).

</details>

## Summary

EBS volumes are needed to give EC2 instances reliable, scalable, and performant disk storage independent from the instance's lifecycle — a critical component for most production applications running on AWS.
