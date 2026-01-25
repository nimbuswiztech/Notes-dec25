# AWS EC2 Scenarios

#### AWS EC2 Scenarios

<details>

<summary>Question 1: In a production setup, your EC2 instance in us-east-1 suddenly stops responding to SSH. You've confirmed the instance is running via console, but no network connectivity. Walk me through your troubleshooting steps and resolution.</summary>

In production, check instance status in the EC2 console for system status checks — reboot if Instance status check fails (host issues). If checks pass:

* Verify VPC security group inbound rules allow SSH (port 22) from your IP or bastion, and NACLs aren't blocking.
* Note ping/ICMP may be blocked; use AWS CLI: `aws ec2 describe-instance-status --instance-ids i-1234567890abcdef0`.
* Check ENI: ensure network interface attached and in correct subnet/AZ.
* Verify route tables (subnet has route to IGW for public IP).
* Inspect VPC Flow Logs to confirm traffic drops.
* Common fixes: update SG rules, reattach network interface, reboot instance.
* Best practice: enable detailed CloudWatch monitoring.

</details>

<details>

<summary>Question 2: You're migrating 50 EC2 instances to a new AWS account. Instances use encrypted EBS volumes. How do you handle this with minimal downtime, including IAM roles and data integrity?</summary>

* Snapshot all EBS volumes: `aws ec2 create-snapshot --volume-id vol-123 --description "pre-migration"`.
* Share and copy snapshots to target account; handle KMS: share KMS key cross-account or re-encrypt during copy (`copy-snapshot`).
* Register AMIs from snapshots in target account: `aws ec2 register-image --name "migrated-ami"`.
* Launch instances from those AMIs in target account.
* Update IAM instance profiles: `aws ec2 associate-iam-instance-profile`.
* Use blue-green approach: run parallel instances and cutover DNS; downtime minimal.
* Validate data integrity with `cryptsetup luksDump` or `fsck` after attach.
* Consider AWS Migration Hub for orchestration.

</details>

<details>

<summary>Question 3: An EC2 instance hits 100% CPU during peak hours, causing app slowness. You've got CloudWatch alarms. How do you scale it out practically?</summary>

* Enable detailed monitoring (1-min granularity).
* Create CloudWatch alarm: CPUUtilization >80% for 2 periods.
* Put instances behind an Auto Scaling Group (ASG) with min/desired/max (e.g., min 2, desired 2, max 10).
* Add target tracking scaling policy (example CLI call provided): `aws autoscaling put-scaling-policy --auto-scaling-group-name my-asg --policy-name scale-out --policy-type TargetTrackingScaling --target-tracking-configuration file://config.json`
* Use a launch template with user data to initialize app.
* Use ELB/ALB health checks.
* In production, scaling from 2→8 reduced CPU to \~40%.

</details>

***

#### VPC and Networking Scenarios

<details>

<summary>Question 4: Your VPC has public and private subnets. Web servers in private subnet need internet outbound only, no inbound from internet. How do configure without NAT Gateway per AZ, considering cost?</summary>

* Deploy a NAT instance in a public subnet (e.g., t3.micro).
* Disable source/dest check: `aws ec2 modify-instance-attribute --instance-id i-123 --source-dest-check '{"Value":false}'`.
* Private subnet route tables: 0.0.0.0/0 → NAT ENI.
* NAT SG: outbound all, inbound SSH only from bastion.
* For HA: run ASG with 2 NAT instances across AZs and script failover.
* Cost trade-off: NAT instance (~~$10/mo) vs NAT Gateway (~~$40+/mo). Monitor and upgrade if egress >1TB.

</details>

<details>

<summary>Question 5: Traffic between two VPCs in same region is dropping despite VPC Peering. What's likely cause and fix?</summary>

* Likely missing routes or SG/NACL blocking.
* Add route in each VPC route table pointing peer CIDR to peering connection (pcx-...).
* Ensure security groups allow traffic from the peer CIDR (use CIDR, not just SG in some setups).
* Remember NACLs are stateless—add explicit return rules.
* If using DNS across peering, enable peering DNS options.
* Troubleshoot with VPC Reachability Analyzer. Fix often by adding SG ingress from peer CIDR.

</details>

<details>

<summary>Question 6: You're setting up a new VPC for prod app. Explain your subnetting strategy for 3 AZs, including CIDR choices and why.</summary>

* Choose /16 VPC (e.g., 10.0.0.0/16) for growth.
* Per AZ allocate:
  * Public /24 each (10.0.1.0/24, 10.0.2.0/24, 10.0.3.0/24) for ALB/NAT.
  * Private app /24 each (10.0.4.0/24, 10.0.5.0/24, 10.0.6.0/24).
  * Private DB /26 each (10.0.7.0/26, etc.) for RDS (smaller range).
* Public subnets route to IGW; private use NAT.
* Tag subnets (e.g., Tier=web). Avoid overlapping on-prem ranges.
* This provides room for scaling to thousands of instances without re-IPing.

</details>

***

#### Security Groups and NACLs

<details>

<summary>Question 7: App servers need DB access (port 3306) but not vice versa. Traffic blocked despite SG allows. Troubleshoot.</summary>

* Verify DB SG inbound allows 3306 from app SG (use SG ID as source).
* Verify app SG outbound allows 3306 to DB SG.
* Confirm both SGs in same VPC.
* Check NACLs for blocking rules on subnets.
* Use: `aws ec2 describe-security-groups --group-ids sg-123`.
* Inspect VPC Flow Logs for REJECTs on port 3306.
* Fix: ensure DB SG inbound includes web SG as source.

</details>

<details>

<summary>Question 8: Audit shows open SG (0.0.0.0/0 SSH). How remediate across 100 instances in prod?</summary>

* Use AWS Config rule (restricted-ssh) to detect.
* Automate remediation with SSM Automation or Lambda: Example SSM call: `aws ssm start-automation-execution --document-name "AWS-UpdateSecurityGroup" --parameters '{"SecurityGroupId":"sg-123","IpPermissions":[{"IpProtocol":"tcp","FromPort":22,"ToPort":22,"IpRanges":[{"CidrIp":"my-ip/32"}]}]}'`
* Tag SGs and trigger Lambda on changes.
* Replace direct SSH exposure with Bastion + Session Manager (SSM) for no open SSH.
* Rollout via automation across SGs to remediate at scale.

</details>

***

#### EBS Scenarios

<details>

<summary>Question 9: Prod EBS gp3 volume at 99% IOPS during backups. How optimize without downtime?</summary>

* Check CloudWatch metrics (BurstBalance, IOPS).
* Modify gp3 volume to provisioned IOPS: `aws ec2 modify-volume --volume-id vol-123 --provisioned-iops 3000 --iops 3000`
* Consider migrating to io2 for consistent performance.
* Snapshot before modify. If needed, attach a new volume from snapshot.
* Stagger backups via Lifecycle Manager to reduce spikes.
* Use EBS-optimized instance types.

</details>

<details>

<summary>Question 10: EC2 terminated accidentally, EBS delete on term enabled. Recover data.</summary>

* Restore from snapshots (if lifecycle/snapshots exist): `aws ec2 describe-snapshots --owner-ids self --filters Name=tag-key,Values=Name`.
* Create volume from snapshot and attach: `aws ec2 attach-volume --volume-id vol-new --instance-id i-new --device /dev/sdf`.
* Mount and recover data (rsync, etc.).
* If no snapshot and volume deleted, recovery is unlikely.
* Disable DeleteOnTermination in launch templates to prevent future loss.

</details>

***

#### ALB and Auto Scaling

<details>

<summary>Question 11: ALB target group shows unhealthy EC2s despite passing local health checks. Fix it.</summary>

* Check ALB target-group health with: `aws elbv2 describe-target-health --target-group-arn arn:...`
* Verify ALB health check path and that target returns 200 on that path.
* Ensure target SG allows traffic from ALB SG (not 0.0.0.0/0) — add ALB SG as allowed source.
* Check protocol mismatch (HTTP vs HTTPS) between ALB and targets.
* Adjust deregistration delay if needed.

</details>

<details>

<summary>Question 12: ASG scales in during low traffic but kills active user sessions. How prevent?</summary>

* Enable connection draining (deregistration delay) on target group (default up to 300s).
* Use ASG lifecycle hooks to pause termination and check session activity; tie to Lambda/SSM to decide when to proceed.
* Use step/target scaling with buffers to avoid aggressive scale-in.
* Implement sticky sessions or session persistence if needed, or move sessions to shared store (Redis) to be stateless.

</details>

<details>

<summary>Question 13: ALB getting 5xx errors from backend during spikes. Architecture fix with ASG?</summary>

* Scale ASG based on RequestCountPerTarget or latency metrics.
* Use target tracking on RequestCountPerTarget > threshold or latency > threshold.
* Make app stateless (disable ALB stickiness) and use RDS Proxy or caching layer.
* Add WAF or throttling to reduce spike-induced overload.

</details>

***

#### Route53 Scenarios

<details>

<summary>Question 14: Route53 health check shows domain down, but app is fine. Prod failover issue?</summary>

* Ensure health check endpoint is reachable publicly (not in private subnet unless using private hosted zone).
* Use Route53 private hosted zone associated with VPC for internal checks when applicable.
* Check health-check configuration (path, port, failure threshold).
* Use secondary endpoint and failover routing policy if needed.
* CLI: `aws route53 get-health-check --health-check-id abc`.

</details>

<details>

<summary>Question 15: Migrate DNS from GoDaddy to Route53 for app.example.com with low TTL. Steps?</summary>

* Lower TTL to 300s one day before migration.
* Create Route53 public hosted zone for domain.
* Create appropriate records (A/CNAME) in Route53.
* Update GoDaddy registrar NS to the 4 Route53 NS records.
* Verify with `dig NS app.example.com`.
* Use weighted/latency routing for blue-green if needed.

</details>

***

#### S3 Scenarios

<details>

<summary>Question 16: S3 bucket used for static site, but uploads from EC2 fail with AccessDenied. IAM fix?</summary>

* Check bucket policy for denies (e.g., deny unencrypted).
* Ensure EC2 IAM role has `s3:PutObject` on `arn:aws:s3:::bucket/*`.
* Apply least privilege and enable versioning/MFA delete if required.
* Example: `aws s3api put-bucket-policy --bucket mybucket --policy file://policy.json`.
* Use OAC or cross-account roles for cross-account access and CloudTrail for audit.

</details>

<details>

<summary>Question 17: S3 lifecycle moves old objects to Glacier, but restore costs spiking. Optimize.</summary>

* Adjust lifecycle: transition to IA after 30d, Glacier after 180d, expire after 1095d.
* Consider Intelligent-Tiering for unpredictable access.
* Monitor with S3 Storage Lens and adjust transitions to reduce restore frequency.
* Exclude frequently-restored prefixes from deep archive.

</details>

***

#### Lambda Scenarios

<details>

<summary>Question 18: Lambda function times out at 15min on large file process from S3 trigger. Scale it.</summary>

* Increase timeout and memory (more memory = more CPU): `aws lambda update-function-configuration --function-name myfunc --timeout 900 --memory-size 3008`
* Use Step Functions to split long tasks into smaller steps.
* Use async processing or queue-based (SQS) decoupling.
* Use EFS if /tmp >512MB.
* Provisioned concurrency for steady traffic.

</details>

<details>

<summary>Question 19: Lambda VPC-attached can't reach S3. Networking fix?</summary>

* Attach a VPC Endpoint for S3: `aws ec2 create-vpc-endpoint --vpc-id vpc-123 --service-name com.amazonaws.region.s3`
* Ensure correct endpoint policy and route tables for subnets using the endpoint.
* Alternatively, ensure NAT/IGW exists in the subnet for internet access.
* VPC endpoints can also reduce egress costs.

</details>

***

#### CloudFormation Scenarios

<details>

<summary>Question 20: CloudFormation stack update fails ROLLBACK due to EC2 in service. Handle it?</summary>

* Use Change Sets to preview changes.
* Include `Capabilities: CAPABILITY_IAM` for IAM resource changes.
* Use Retain policy for critical resources to avoid deletion on rollback.
* Use drift detection: `aws cloudformation detect-stack-drift --stack-name mystack`.
* Use nested stacks and CI/CD (CodePipeline) for safer updates.

</details>

<details>

<summary>Question 21: Template creates VPC + subnets + IGW. Make it reusable across accounts.</summary>

* Parameterize CIDR, AZs, and environment (Env parameter).
* Use Mappings for AZ names per region.
* Add Outputs (VPCId, SubnetIds) for reuse.
* Use Conditions (e.g., IsProd) to vary resources.
* Store template in version control and reuse via CodePipeline/CloudFormation StackSets.

</details>

***

#### Integrated Scenarios

<details>

<summary>Question 22: Prod app: ALB -> ASG(EC2) -> RDS in private VPC subnet. EC2 can't reach RDS. Full troubleshoot.</summary>

* Verify private subnet routes and NAT if needed.
* Confirm RDS SG inbound allows 3306 from EC2 SG.
* Confirm EC2 SG outbound allows 3306 to RDS.
* Ensure subnet groups and AZ placement for RDS are correct.
* Check Secrets Manager for correct credentials.
* Fix typically by referencing EC2 SG in RDS SG inbound.

</details>

<details>

<summary>Question 23: S3 trigger Lambda scales ASG on file count. Cost/security optimize.</summary>

* Use EventBridge or SQS batching to control invocation rate.
* Add DLQ for failed events.
* Apply least-privilege IAM (e.g., `s3:GetObject`).
* Restrict Lambda concurrency (reserved concurrency).
* Monitor costs; example: reserved concurrency and batching reduce costs.

</details>

<details>

<summary>Question 24: CloudFormation deploys EKS-like: VPC, EC2 ASG, ALB. Auto-remediate failed resources?</summary>

* Use custom resource Lambdas for additional checks and remediation.
* Apply Stack Policies to protect resources during updates.
* Use AWS Config drift detection and remediation runbooks.
* Integrate with pipelines for automated retries and validations.

</details>

<details>

<summary>Question 25: Multi-account: Route53 private zone resolves across VPCs. S3 backend via Lambda in spoke VPC.</summary>

* Share private hosted zone via AWS RAM to other accounts.
* Use VPC peering or Transit Gateway for VPC connectivity; add VPC endpoints for S3 access.
* Ensure endpoint policies and route tables are configured.
* Transit Gateway recommended for scale across multiple accounts.

</details>
