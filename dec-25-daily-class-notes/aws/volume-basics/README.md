# volume basics

We need **EBS (Elastic Block Store) volumes in AWS** because they provide **durable, persistent, block-level storage** that can be attached to EC2 instances, functioning like a hard drive for storing data that requires frequent updates or continuous access. EBS volumes are essential for operating systems, databases, application code, logs, and throughput-intensive or I/O-sensitive workloads. They persist independently from the life of the instance, so data remains intact even if the instance stops or terminates.

Volumes in AWS, specifically **Amazon EBS (Elastic Block Store) volumes**, are durable, block-level storage devices that you attach to Amazon EC2 instances to provide persistent storage, similar to a physical hard drive attached to a computer.

## Key aspects of AWS EBS volumes

* **Block-level storage:** These volumes appear to the EC2 instance like a hard drive, allowing you to store files, install applications, or host databases.
* **Persistence:** Data on an EBS volume persists independently of the EC2 instance’s lifecycle. This means your data remains intact even if the instance is stopped or terminated.
* **Flexibility and scalability:** You can dynamically modify size, performance (IOPS), and volume type on live volumes without downtime, making them adaptable to changing workload requirements.
* **Multiple volume types:** AWS offers a variety of EBS volume types optimized for different use cases:
  * SSD-backed volumes (General Purpose SSD gp2/gp3, Provisioned IOPS SSD io1/io2) for transactional workloads requiring low latency and high IOPS.
  * HDD-backed volumes (Throughput Optimized HDD st1, Cold HDD sc1) for throughput-intensive or infrequently accessed workloads.
* **High availability and durability:** Data on EBS volumes is automatically replicated within the same Availability Zone to protect against hardware failures and ensure durability, with some volume types providing up to 99.999% durability.
* **Backup and recovery:** You can create snapshots of EBS volumes, which are incremental backups stored in Amazon S3, enabling easy restoration or migration of data.
* **Security:** EBS volumes support encryption using AWS Key Management Service (KMS), protecting data at rest and in transit between instances and volumes.

## Key reasons why EBS volumes are needed in AWS

* **Persistent storage:** Unlike instance store volumes that are ephemeral and lost when an instance stops or terminates, EBS volumes retain data independently, enabling reliable long-term storage for EC2 instances.
* **Flexibility and scalability:** You can dynamically resize volumes, adjust performance (such as IOPS), and change volume types on live production workloads. This scalability supports various application needs, from boot volumes to large databases.
* **Performance tailoring:** Multiple volume types (General Purpose SSD, Provisioned IOPS SSD, Throughput Optimized HDD, Cold HDD) let you select storage optimized for cost, throughput, or IOPS depending on your workload.
* **Data availability and durability:** EBS volumes replicate data within an Availability Zone to prevent hardware failure data loss, ensuring high availability and durability.
* **Multiple attachments and shared storage:** Some volume types support Multi‑Attach, allowing a single volume to be attached to multiple Nitro‑based instances, facilitating shared data access.
* **Use cases:** EBS volumes are suitable for storing operating system files, application code, databases, logs, temporary files, and throughput‑intensive workloads which need persistent, high‑performance storage on AWS.

<details>

<summary>Contrast with other AWS storage services</summary>

While other AWS storage services like S3 provide object storage, and RDS offers managed databases, EBS volumes uniquely provide block-level, persistent storage that behaves like a traditional disk attached to an EC2 instance, suitable for OS boot drives and workloads requiring frequent random I/O.

</details>

{% hint style="info" %}
In summary: EBS volumes provide EC2 instances with reliable, scalable, and performant disk storage that persists independently from the instance lifecycle—making them critical for most production applications on AWS.
{% endhint %}
