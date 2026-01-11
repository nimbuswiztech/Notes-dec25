# aws scenario question

{% stepper %}
{% step %}
### Part A — VPC and Network Design

a) Subnets / AZs

* Use a minimum of 2 AZs for high availability (preferably 3 if budget allows).
* Create 6 subnets total: in each of 2 AZs create:
  * 1 public subnet for the web tier (AZ-A public, AZ-B public)
  * 1 private application subnet for the app tier (AZ-A app, AZ-B app)
  * 1 private database subnet for the DB tier (AZ-A db, AZ-B db)\
    Reasoning: Multi-AZ spread provides HA for the web and app tiers and allows DB redundancy (e.g., multi-AZ RDS or replicas) without exposing DB to the internet.

b) CIDR plan (VPC /16)

* VPC: 10.0.0.0/16
* Subnet allocation (example /24s to leave room for growth):
  * Public AZ-A: 10.0.1.0/24
  * Public AZ-B: 10.0.2.0/24
  * App AZ-A: 10.0.11.0/24
  * App AZ-B: 10.0.12.0/24
  * DB AZ-A: 10.0.21.0/24
  * DB AZ-B: 10.0.22.0/24

c) Communication and DB protection

* Public subnets host NAT Gateways and ALBs / NAT as needed; only web tier instances or ALB have public IPs.
* App and DB tiers live in private subnets (no public IPs).
* Use Security Groups to allow traffic from web -> app and app -> db only (SG references). No inbound route from internet into private subnets for DB. Public route table routes 0.0.0.0/0 to Internet Gateway only for public subnets.

d) Additional VPC components

* Internet Gateway (IGW): attach to the VPC and associate with public route table so public subnets have internet access.
* NAT Gateway(s): in each public subnet for outbound internet access from private subnets (or use a NAT instance if required). One per AZ for resilience.
* Route tables: separate route table for public and private subnets (private route tables route 0.0.0.0/0 to NAT Gateway).
* NACLs: optional; keep default or implement conservative stateless rules if needed. Primary filtering should be Security Groups (stateful).
* VPC Endpoints: add Gateway VPC Endpoint for S3 and/or Interface Endpoints for other AWS APIs if you want private access from private subnets without NAT.
{% endstep %}

{% step %}
### Part B — Security Groups Configuration

a) SG-WEB (Web tier)

* Inbound:
  * HTTP (80) and HTTPS (443) from 0.0.0.0/0 (and ::/0 for IPv6 if used).
  * (If using ALB, the ALB's SG will be the public-facing SG and web instances may only accept traffic from the ALB SG.)
* Outbound:
  * Allow to SG-APP on required ports (e.g., all TCP or specific app ports). Typically allow outbound to SG-APP: all (or app port range).
* Purpose: expose web to internet while only initiating traffic to app tier.

b) SG-APP (Application tier)

* Inbound:
  * Allow from SG-WEB only on the application port(s) (e.g., 443 or app port). Using SG IDs ensures only traffic from web tier is accepted.
* Outbound:
  * Allow to SG-DB on DB port(s) (e.g., 3306 for MySQL) and to required external endpoints (e.g., S3 endpoints) as needed.
* Notes: Do not open app inbound to 0.0.0.0/0.

c) SG-DB (Database tier)

* Inbound:
  * Allow only from SG-APP on the DB port (e.g., 3306 for MySQL, 5432 for Postgres). No inbound from internet.
* Outbound:
  * Minimal; ideally restrict outbound to required management or backup endpoints. Often allow only necessary outbound traffic or none.
* Principle: least privilege — only permit the minimal source ports and sources required.

d) Advantage of using Security Group IDs as sources

* Security group references are dynamic and follow auto-scaled instances (no need to manage changing IPs). They preserve intended trust relationships even when instances scale up/down or receive new IPs, reducing operational complexity and attack surface.
{% endstep %}

{% step %}
### Part C — EC2 Instances and Volumes

a) Instance types (recommendations)

* Web servers: t3 / t4g (burstable) for cost-sensitive workloads that are not CPU-bound; m5 / m6 for steady medium load. Choose instance size (t3.medium, t3.large, m5.large) based on expected CPU/memory and network needs.
* App tier: m5/m6 if moderate CPU/memory; c5/c6 for CPU-bound workloads.
* DB server: r5/r6 (memory-optimized) or instance class required by managed DB (RDS) depending on DB workload; if self-managed, consider I/O throughput and network.

b) Volume types per data requirement

* Root volumes (100 GB per web instance): gp3 (general purpose SSD) — cost-effective with configurable IOPS and throughput. Use gp3 to meet performance baseline.
* Shared application data (50 GB): Amazon EFS (Elastic File System) — NFS-based shared filesystem mounted to all 3 web instances; scales and provides shared access. If very small and only occasional sharing, could use an S3-backed approach, but EFS is simplest for POSIX file sharing.
* Database backups (200 GB persisted beyond instance lifecycle): Store backups in S3 (durable, cost-effective). Use EBS snapshots as intermediate durable point-in-time backups; copy snapshots to S3 (cross-region snapshot copy) or use lifecycle policies.
* Database main volume (high I/O): io2 (or io1 if needed) EBS volumes for provisioned IOPS and high durability. io2 provides high IOPS/throughput and durability for production DB.

c) Shared storage for application data

* Use Amazon EFS mounted on all web instances across AZs (ensure EFS mount targets in each AZ). EFS provides POSIX semantics and concurrent access across instances.

d) Database server volume configuration

* Use io2 EBS volumes sized and provisioned with required IOPS (provisioned IOPS matching DB needs). Consider:
  * Separate EBS volumes for data, logs, and temp to optimize I/O patterns.
  * Use multiple volumes and striping (mdadm) if needed for throughput, or use instance storage if ephemeral I/O is acceptable (not for durability).
  * For managed RDS, use Provisioned IOPS (io2) and enable Multi-AZ.

e) Backup & disaster recovery for volumes

* Regular automated EBS snapshots (use Data Lifecycle Manager for retention/lifecycle).
* Copy snapshots to another region for DR (cross-region snapshot copy).
* Store periodic logical DB backups to S3 (e.g., mysqldump / pg\_dump) and/or use native DB backup features.
* Use lifecycle policies to transition older backups to cheaper storage (S3 Glacier) if appropriate.
* Test restore runbooks periodically.
{% endstep %}

{% step %}
### Part D — Advanced: Expanding an in-use 500 GB io2 volume (85% used)

a) Available options

* Modify the EBS volume (increase size) — for modern EBS (gp3/io2) you can generally modify size and IOPS online without detaching; after modification, you often need to grow the partition and filesystem on the instance (online grow supported by many filesystems). Some older volume types or OS configurations may require a reboot.
* Create a snapshot and restore to a new larger volume, attach to a standby/replica instance, sync and cutover.
* Create a new larger volume from snapshot and then replace/migrate with minimal downtime (attach, rsync, failover).
* If IOPS are the constraint, scale IOPS up on io2 or move to more performant instance type.

b) Zero-downtime approach

* Preferred zero-downtime: Increase the volume size in-place (ModifyVolume API), then grow the filesystem online. Steps:
  1. Create snapshot (safety).
  2. Call ModifyVolume to increase size (and IOPS if needed). AWS supports online expansion for many setups.
  3. On the instance, rescan the block device and grow the partition (if partitioned) and then grow the filesystem (e.g., xfs\_growfs or resize2fs) — these can often be done online without unmounting.
* If in-place resize cannot be performed without downtime, alternative zero-downtime method: create a new larger EBS volume from a recent snapshot, attach it to a replica DB or read replica, sync changes (using replication or rsync), then promote replica and switch traffic.

c) Precautions before executing

* Take a snapshot before any change.
* Validate that the instance OS and filesystem support online expansion; have documented commands for filesystem growth.
* Ensure backups are recent and functional (test restore in non-prod).
* Confirm IAM permissions and current CloudWatch metrics.
* Inform stakeholders and schedule maintenance window if risk is non-zero.
* If using replication-based migration: verify replication lag is low and application can redirect connections.

d) Monitoring the operation

* Monitor CloudWatch metrics: VolumeThroughput, VolumeWriteOps, VolumeReadOps, VolumeIdleTime, VolumeConsumedReadWriteOps, BurstBalance (if applicable).
* Monitor instance-level CPU, disk queue length, and I/O wait.
* Monitor application logs and DB health metrics (latency, error rates, replication lag).
* Track ModifyVolume progress via AWS API/console and CloudTrail events.
* Alert on anomalies during/after the resize.
{% endstep %}

{% step %}
### Part E — Troubleshooting

a) Web accessible from office but not home — troubleshooting steps

1. Verify client differences: test from home with curl/traceroute to public IP/ALB.
2. Check DNS resolution (does name resolve to the correct public IP/ALB?).
3. Inspect ALB / public IP health and listener configuration.
4. Check Security Groups on ALB and web instances — ensure 0.0.0.0/0 is allowed for 80/443 (or source ranges include home).
5. Verify Network ACLs (NACLs) for public subnet are not blocking home IP ranges (NACLs are stateless and can block unexpectedly).
6. Check route table for public subnet points to IGW.
7. Check corporate office may be on a cached route or using a VPN; check if home ISPs block certain ports.
8. Use VPC Flow Logs and ALB access logs to see if requests arrive and are dropped or never reach VPC.

b) Likely cause for S3 backup failures from private DB subnet

* Private subnet lacks internet access (no NAT Gateway) and there is no S3 VPC endpoint, so instance cannot reach S3 public endpoints.

c) Resolve backup issue while maintaining security

* Option A (cost effective & secure): Create a Gateway VPC Endpoint for S3 and update private subnet route tables. This allows private instances to access S3 without traversing the internet or NAT.
* Option B: Provision NAT Gateway(s) in public subnets (one per AZ for HA) to allow outbound internet to S3. Use route tables to send 0.0.0.0/0 from private subnets to NAT.
* For extra security, use endpoint policies to restrict S3 access to specific buckets and IAM roles. Use VPC endpoint policies and IAM policies to limit access.

d) Monitoring to prevent future issues

* Enable CloudWatch metrics and alarms for NAT Gateway throughput/cost and VPC Endpoint metrics.
* Enable VPC Flow Logs to detect dropped traffic and identify blocked sources/destinations.
* Monitor S3 backup job success/failure metrics and send alerts on failures.
* Enable ALB access logs and CloudWatch alarms for 5xx errors and unhealthy targets.
* Implement synthetic checks (health checks) from multiple external locations.
{% endstep %}

{% step %}
### Bonus Challenge — Disaster Recovery (RTO 15m, RPO 1h, minimize secondary region cost)

Design (summary of components and approach)

* Use cross-region snapshot copies and AMI copies for quick provisioning in target region.
* For primary: enable frequent automated EBS snapshots (every hour or less) and replicate snapshots to secondary region (cross-region snapshot copy).
* For compute: automate AMI copy of web/app images to secondary region as part of CI/CD so instances can be launched quickly.
* For database:
  * If using RDS: enable automated snapshots and cross-region read replica that can be promoted (if budget allows). For lower cost, keep asynchronous cross-region read replica turned off and use snapshots for restore; but to meet 15m RTO you likely need a warm standby or a pre-provisioned minimal fleet in DR region.
  * Warm-standby approach (cost-balanced): run small-sized instances or RDS in secondary region (minimal capacity) continuously and scale up on failover. Maintain continuous replication (read replica) where possible to meet RPO \~1 hour (or less).
* Networking & data:
  * Replicate S3 buckets with cross-region replication for object data.
  * Use Route53 health checks and weighted/Failover routing to quickly switch DNS to DR region.
* Orchestration:
  * Automate failover runbook via infrastructure-as-code (CloudFormation / Terraform) and Lambda to orchestrate promotion of snapshots/AMI and scaling. Pre-bake AMIs to reduce provisioning time.
* Cost trade-offs:
  * Cold standby (cheapest): only store snapshots/AMIs and spin up infra on failover; likely misses 15m RTO.
  * Warm standby (balanced): minimal instances running in secondary region to reduce provisioning time and meet short RTO at moderate cost.
  * Pilot light (minimal core services continually running + automated scripts to scale) is common compromise.
* Testing:
  * Regular DR drills: timed runbooks to restore from snapshots to confirm RTO/RPO.
  * Automate creation of test environments from DR artifacts and validate apps and DB integrity.
  * Validate DNS failover and application end-to-end checklists.

How to achieve RTO 15m / RPO 1h within budget

* Maintain continuous cross-region incremental snapshots (RPO <= 1h).
* Keep a pilot-light (small, continuously running critical components) in secondary region so point-in-time restoration and scaling can be completed within 15 minutes.
* Pre-copy AMIs and snapshot-to-volume automation to reduce provisioning time.
* Use Route53 failover/health checks for rapid DNS switch.

Testing approach

* Schedule quarterly DR tests: run timed failover exercises from snapshots/AMIs to ensure full restore within target RTO and validate data within RPO.
* Automate smoke tests post-failover to confirm functionality.
{% endstep %}
{% endstepper %}
