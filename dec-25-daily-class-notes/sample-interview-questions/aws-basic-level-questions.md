# AWS Basic Level Questions

{% stepper %}
{% step %}
### In a production VPC setup, you've launched an EC2 instance but it can't reach the internet. What do you check first, and how do you fix it?

* Verify the instance is in a public subnet with a route to an Internet Gateway (IGW) attached to the VPC.
* Check route tables: `aws ec2 describe-route-tables --route-table-ids rtb-xxx`.
* If missing, associate the subnet to a public route table with route `0.0.0.0/0 -> IGW`.
* Ensure the instance has a public IP or Elastic IP, and that Security Groups and NACLs allow outbound HTTP/HTTPS.
* Common gotcha: instances in private subnets require a NAT Gateway for outbound internet access.
* Remediation example: add a NAT for private subnet traffic; connectivity restored immediately.
{% endstep %}

{% step %}
### How do you attach an EBS volume to a running EC2 instance across AZs, and what happens if the instance fails over?

* Attach to running instance (if supported):\
  `aws ec2 attach-volume --volume-id vol-xxx --instance-id i-xxx --device /dev/sdf`\
  Inside instance: `mkfs -t ext4 /dev/xvdf; mount /dev/xvdf /data`
* EBS volumes are AZ-specific. For cross-AZ failover: create a snapshot and restore a new volume in the target AZ:\
  `aws ec2 create-snapshot --volume-id vol-xxx`
* For shared access on Nitro instances, EBS Multi-Attach can be used.
* For failover resilience of stateful data, use ASG patterns with EFS or replicate via snapshots.
{% endstep %}

{% step %}
### Explain how Security Groups act as a virtual firewall in a VPC with multiple tiers (web, app, DB). Give a real config example.

* Security Groups (SGs) are stateful: return traffic is allowed automatically.
* 3-tier example:
  * Web SG: allow inbound 80/443 from 0.0.0.0/0; outbound to App SG on 8080.
  * App SG: inbound from Web SG only; outbound to DB SG on 3306.
  * DB SG: inbound from App SG only.
* CLI example to allow web ingress:\
  `aws ec2 authorize-security-group-ingress --group-id sg-web --protocol tcp --port 80 --cidr 0.0.0.0/0`
* Best practice: least privilege and reference SGs (not CIDRs) for autoscaling environments.
{% endstep %}

{% step %}
### Your S3 bucket is public by mistake in prod. How do you secure it immediately, and prevent recurrence?

* Inspect ACL: `aws s3api get-bucket-acl --bucket mybucket`
* Block public access immediately:\
  `aws s3api put-public-access-block --bucket mybucket --public-access-block-configuration BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true`
* Apply a restrictive bucket policy (deny public actions), e.g.:\
  {"Statement":\[{"Effect":"Deny","Principal":"_","Action":"s3:_","Resource":"arn:aws:s3:::mybucket/\*","Condition":{"StringNotEquals":{"aws:PrincipalAccount":"123456789012"\}}}]}
* Use SCPs, IAM Access Analyzer, and automated audits to prevent recurrence.
{% endstep %}

{% step %}
### Walk through creating a simple CloudFormation stack for an EC2 instance with EBS. What's a common deployment pitfall?

*   Template snippet:

    ```
    Resources:
      EC2Instance:
        Type: AWS::EC2::Instance
        Properties:
          ImageId: ami-xxx
          InstanceType: t3.micro
          BlockDeviceMappings:
            - DeviceName: /dev/xvda
              Ebs:
                VolumeSize: 20
                VolumeType: gp3
    ```
* Deploy: `aws cloudformation create-stack --stack-name myec2 --template-body file://template.yaml`
* Common pitfall: not specifying DeletionPolicy for EBS; use `DeletionPolicy: Retain` to avoid data loss on stack delete.
* Production best practices: parameterize AMI/InstanceType and use ChangeSets for previews.
{% endstep %}

{% step %}
### ALB health checks are failing for your web app, causing all targets to be unhealthy. Troubleshooting steps?

* Check target health: `aws elbv2 describe-target-health --target-group-arn arn-xxx`
* Verify health check path, matcher, and expected status (e.g., `/health` returns 200).
* Check Security Group allows health check port, and app is responding on that path/port.
* Common issues: protocol mismatch (HTTP vs HTTPS), too-short timeout, wrong path or matcher.
* Example fix: set matcher to 200-299 and use correct path (e.g., `/actuator/health` for Spring Boot).
{% endstep %}

{% step %}
### Design a VPC for a microservices app with public web, private app/DB. Include NACLs vs SGs differences in prod.

* Architecture:
  * Public subnets: ALB and web; route via IGW.
  * Private subnets: app and DB; NAT Gateway for outbound.
* Security controls:
  * Security Groups are stateful, applied to ENIs (instance-level).
  * NACLs are stateless, applied to subnets (explicit allow/deny for return traffic).
* Example NACLs: allow 80/443 inbound to public subnet and ephemeral ports 1024-65535 outbound; DB NACL allow 3306 only from app subnet CIDR.
* Use NACLs for additional compliance/hardening (e.g., PCI) to limit lateral movement.
{% endstep %}

{% step %}
### EC2 Auto Scaling Group (ASG) isn't scaling out during traffic spike. What metrics/configs do you audit?

* Check CloudWatch alarms (CPU, request count, custom metrics).
* Verify ASG min/max/desired: `aws autoscaling describe-auto-scaling-groups --auto-scaling-group-names myasg`
* Confirm scaling policies are attached and correct (target tracking vs step scaling).
* Review cooldown periods and evaluation periods.
* Example fix: add target tracking on ALB request count (TargetValue=100 requests/instance) and tune cooldown.
* Also check account/region limits (default 1000 instances per ASG).
{% endstep %}

{% step %}
### Route53 for a failover setup: primary site down, traffic not switching. How to configure active-active vs active-passive?

* Use health checks on endpoints.
* Active-passive: primary has failover routing (PRIMARY/SECONDARY); Route53 routes to secondary when primary fails.
* Active-active: use weighted/latency routing with health checks; unhealthy endpoints are removed.
* Apply via CLI: `aws route53 change-resource-record-sets --hosted-zone-id Z1A --change-batch file://failover.json` (include Failover and HealthCheckId).
* Production: latency-based with geo-failover reduces failover time significantly.
{% endstep %}

{% step %}
### S3 lifecycle policy deleted critical data prematurely. How to recover and best practices for versioning?

* Enable versioning: `aws s3api put-bucket-versioning --bucket mybucket --versioning-configuration Status=Enabled`
* Recover objects: list versions `aws s3api list-object-versions --bucket mybucket` and restore needed versions (remove delete markers):\
  `aws s3api delete-object --bucket mybucket --key file.txt --version-id xxx`
* Best practices: enable versioning, use MFA Delete for critical buckets, and use S3 Inventory for audits. Configure lifecycle transitions/expirations carefully (e.g., transition to IA after 30 days).
{% endstep %}

{% step %}
### Lambda cold starts are killing your API perf (500ms+). Mitigation for a Node.js function behind API Gateway?

* Use Provisioned Concurrency:\
  `aws lambda put-provisioned-concurrency-config --function-name myfunc --qualifier $LATEST --provisioned-concurrency-config ProvisionedConcurrentExecutions=5`
* Optimize bundle size (esbuild), initialize heavy resources outside the handler, minimize startup work.
* Tune memory (increases CPU) and consider warming strategies.
* Limits: 10 GB memory, 15 min timeout. Example: p99 latency dropped from 1.2s to 200ms after provisioned concurrency and bundle optimization.
{% endstep %}

{% step %}
### CloudFormation stack update rolls back on EC2 UserData failure. How to debug and make it idempotent?

* Inspect CloudFormation events: `aws cloudformation describe-stack-events --stack-name mystack`
*   Make UserData idempotent, e.g.:

    ```
    #!/bin/bash
    yum update -y
    if ! rpm -q nginx; then yum install -y nginx; fi
    ```
* Use cfn-init and cfn-signal to report success/failure and use WaitCondition for ordering.
* Wrap provisioning commands to handle retries and partial failures.
{% endstep %}

{% step %}
### Multi-AZ ALB with ASG: targets in private subnets unhealthy. Cross-zone load balancing off? Explain impact.

* If cross-zone load balancing is disabled, ALB sends traffic only to targets in the same AZ as the incoming request. Under AZ skew or instance drain, traffic can be uneven and cause apparent unavailability.
* Enable cross-zone load balancing:\
  `aws elbv2 modify-load-balancer-attributes --load-balancer-arn arn-xxx --attributes Key=load_balancing.cross_zone.load_balancing.enabled,Value=true`
* Production impact: enabling ensured even distribution during an AZ drain.
{% endstep %}

{% step %}
### EBS performance tanked on gp3 volume during DB workload. Optimization and monitoring?

* gp3 baseline offers certain IOPS/throughput; provision higher IOPS/throughput if needed:\
  `aws ec2 modify-volume --volume-id vol-xxx --iops 12000 --throughput 500`
* Monitor CloudWatch: VolumeReadOps, VolumeWriteOps, Throughput, BurstBalance.
* For very high IO, use io2 or io2 Block Express.
* Example: migrated to io2 BlockExpress in prod and halved query latency.
{% endstep %}

{% step %}
### VPC peering between prod and dev accounts: traffic blocked. Transit Gateway vs peering limits?

* Verify route tables include the peering connection (pcx-xxx), and Security Groups/NACLs allow traffic.
* Peering limitations: no transitive routing and cannot peer overlapping CIDRs.
* For hub-and-spoke or many VPCs, use Transit Gateway (TGW): `aws ec2 create-transit-gateway`
* Production: TGW with RAM sharing scaled to multiple VPCs and avoided peering complexity.
{% endstep %}

{% step %}
### S3 cross-account replication lagging, data inconsistency. Setup with CRR best practices.

* Ensure source/destination buckets have versioning enabled and replication role/policy allows `s3:ReplicateObject`.
* Monitor ReplicationStatus metric and ReplicationTime metrics; alert on Pending/FAILED.
* Use same-region replication for lower latency where possible; consider KMS-key permissions and rotation delays.
* Production: added metrics alarms on replication failures and switched to same-region replication to reduce lag.
{% endstep %}

{% step %}
### Lambda@Edge for global content personalization: origin response trigger slow. Debugging?

* Check CloudWatch logs (CloudFront logs and Lambda@Edge logs) and use Logs Insights to find latency hotspots.
* Limits: response size \~1MB, execution time constraints; reduce processing in Origin Response.
* Optimize by caching personalization where possible, using ViewerRequest, and minimizing origin hits with CloudFront behaviors.
* Production: caching headers in ViewerRequest cut origin requests by \~80%.
{% endstep %}

{% step %}
### Auto Scaling lifecycle hooks not firing during scale-in, instances terminate uncleanly.

* Ensure lifecycle hook exists for `autoscaling:EC2_INSTANCE_TERMINATING`:\
  `aws autoscaling put-lifecycle-hook --lifecycle-hook-name myhook --auto-scaling-group-name myasg --lifecycle-transition autoscaling:EC2_INSTANCE_TERMINATING --notification-target-arn arn-sns`
* Configure notification target (SNS/SQS/Lambda) and complete lifecycle action programmatically (`CompleteLifecycleAction`).
* Use hooks to drain connections (e.g., via ALB deregistration) before termination.
* Production: used hooks to drain ALB connections and ensure graceful shutdown.
{% endstep %}

{% step %}
### CloudFormation nested stacks for modular infra: circular dependency error on update.

* Use `AWS::CloudFormation::Stack` resources, avoid circular exports/Imports across nested stacks.
* Break coupling by passing parameters or storing cross-stack values in SSM Parameter Store or using Outputs with `Export`.
* Use DependsOn where appropriate to control ordering, but prefer decoupling via exported values.
* Production pattern: service catalog + nested stacks, export ARNs to SSM for sharing.
{% endstep %}

{% step %}
### Route53 private hosted zone with on-prem VPN: DNS resolution fails post-failover.

* Ensure the private hosted zone is associated with the relevant VPCs.
* Use Route53 Resolver endpoints (Inbound/Outbound) and conditional forwarding rules to support hybrid DNS.
* For hybrid setups, integrate Resolver endpoints across TGW connections or VPN, and sync AD if needed.
* Production: used Route53 Resolver endpoints across TGW to restore DNS resolution.
{% endstep %}

{% step %}
### Prod EC2 fleet 503s from ALB despite healthy targets. Deregistration delay?

* Deregistration delay (connection draining) defaults to 300s, which can keep connections open and cause 503s during deploys.
* Reduce deregistration delay on target group to a lower value (e.g., 30s) if appropriate, and ensure graceful draining is implemented.
* Production change: reduced delay and used draining to avoid request pile-up.
{% endstep %}

{% step %}
### S3 static site with Lambda@Edge OAI: unauthorized 403s globally.

* Root cause often missing CloudFront Origin Access Identity (OAI) permission in bucket policy or incorrect principal.
* Ensure bucket policy allows CloudFront OAI principal and restricts direct public access.
* Optionally use WAF for geo-blocking/filters.
* Fix: add correct OAI principal to bucket policy; 403s resolved.
{% endstep %}

{% step %}
### CloudFormation drift detected on ASG min-size after manual CLI change. Remediation?

* Detect drift: `aws cloudformation detect-stack-drift --stack-name mystack`
* Remediate by updating the stack to match the desired state or import the manual change into the template.
* Enforce via AWS Config rules and EventBridge automation to revert or notify on drift.
* Production: implemented auto-remediation for critical drift types.
{% endstep %}

{% step %}
### VPC Flow Logs show dropped DB traffic despite SG ok. NACL ephemeral ports?

* NACLs are stateless; ensure return traffic ephemeral port range (1024-65535) is allowed.
* Add corresponding allow rules for ephemeral ports and DB ports in both inbound and outbound directions on the subnet NACLs.
* After adding rules, traffic flowed correctly.
{% endstep %}

{% step %}
### High Lambda concurrency throttled at 1000; ASG alternative for steady load?

* Lambda has concurrency limits; set reserved concurrency per function or request quota increases.
* For steady high throughput, consider container-based solutions (ECS Fargate / EC2 ASG) which can provide predictable capacity and cost profile.
* Production: migrated batch/steady workloads to ECS, reducing cost by \~40% and avoiding Lambda throttling.
{% endstep %}
{% endstepper %}
