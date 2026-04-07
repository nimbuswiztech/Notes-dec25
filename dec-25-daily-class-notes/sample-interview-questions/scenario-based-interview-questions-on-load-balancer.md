# Scenario based interview questions on Load Balancer

{% stepper %}
{% step %}
All targets unhealthy after deployment

**Question:** You deployed a new version of your app behind an ALB, and suddenly all targets in the target group show as “unhealthy”. How would you troubleshoot and fix this?

**Answer:**

* Check the target group’s health check configuration: protocol, port, path, thresholds, timeout, and interval. Confirm the health-check path is actually served by the new app version (e.g., `/health` vs `/`).
* From one of the instances, curl the health-check URL on the application port (not via ALB) to confirm HTTP 200 and expected response time; if this fails, fix app/listener binding or web server config.
* Validate security groups: ALB SG must allow outbound to target port, and instance SG must allow inbound from the ALB SG on the health check port; avoid using `0.0.0.0/0` unnecessarily.
* If the app is slow under load, increase HealthCheck timeout/interval/thresholds so transient spikes don’t mark instances unhealthy, and consider scaling out.
{% endstep %}

{% step %}
### Users see intermittent 5xx from ALB

**Question:** In production, users intermittently see 502/504 errors via an ALB. What’s your debugging approach?

**Answer:**

* Check ALB metrics in CloudWatch (HTTPCode\_ELB\_5XX\_Count, TargetResponseTime, HealthyHostCount). If 5xx increases with higher latency, it usually indicates backend saturation or timeouts.
* Enable ALB access logs for a short period and sample failed requests (timestamp, client IP, target IP, target status) to see if a specific AZ/instance or path correlates.
* Validate target timeouts and connection limits (web server, app server, DB) and align them with ALB idle timeout; for long requests, increase ALB idle timeout above typical request time.
* If only one target group or path is affected, focus on that microservice; scale it (ASG), optimize queries, or add caching to reduce backend pressure.
{% endstep %}

{% step %}
### Client timeouts vs ALB idle timeout

**Question:** A team reports clients getting disconnected or seeing “client closed connection” in logs when using ALB. What could be happening and how do you fix it?

**Answer:**

* ALB can close connections after its idle timeout; clients closing earlier will show up as client disconnects.
* Compare client timeout, ALB idle timeout, and backend response times; ensure the client timeout is greater than or aligned with ALB idle timeout and that the backend can respond within that window.
* For long-running operations (file exports, reports), either increase ALB idle timeout or redesign to async processing (job queue + polling) so the HTTP call remains short.
{% endstep %}

{% step %}
### One AZ overloaded, others underused

**Question:** You notice EC2 instances in one AZ are getting significantly more traffic than others behind an ALB. What could cause this and how do you correct it?

**Answer:**

* Check whether cross-zone load balancing is enabled on the ALB; if disabled and instance counts differ per AZ, you’ll see skewed load.
* Either enable cross-zone load balancing to even out traffic or balance instance counts per AZ using ASG with balanced subnets.
* Confirm all enabled AZs are registered with the ALB and have healthy targets; unhealthy targets or disabled AZs push more traffic to remaining zones.
{% endstep %}

{% step %}
### Simple TCP app: ALB vs NLB

**Question:** You have a high-throughput, low-latency TCP-based service (no HTTP) exposed to external customers. Would you choose ALB or NLB and why?

**Answer:**

* Prefer NLB: it operates at layer 4 with very low latency, supports millions of connections per second, and is designed for TCP/UDP workloads.
* NLB preserves the client source IP, simplifying firewall rules and security monitoring.
* For heavy TCP/UDP throughput, NLB is typically more cost-effective given its connection-based charging model.
{% endstep %}

{% step %}
### Blue/green behind a single ALB

**Question:** How would you implement blue/green deployments using a single internet-facing ALB?

**Answer:**

* Create separate target groups: one for blue (current prod) and one for green (new version); both can run in parallel on different ASGs.
* Use listener rules (host/path conditions), or route traffic gradually to green using weighted routing patterns implemented via listener rules or by managing percentages at the application/proxy layer. Start small, validate, then shift 100%.
* If rollback is needed, flip rules back to blue quickly; DNS and certificates remain unchanged, minimizing blast radius.
{% endstep %}

{% step %}
### Locking down backend instances

**Question:** How do you secure backend EC2 instances so they only accept traffic from the ALB?

**Answer:**

* Put ALB in its own security group (e.g., sg-alb) and attach another SG to instances (e.g., sg-app). In sg-app, allow inbound on app ports only from sg-alb as the source.
* Do not open instance ports to 0.0.0.0/0; restrict SSH/RDP via bastion or SSM, and keep backend instances in private subnets.
* Optionally use AWS WAF with ALB for L7 protection and enforce TLS termination on ALB plus TLS from ALB to instance for end-to-end encryption when required.
{% endstep %}

{% step %}
### Enforcing HTTPS with ALB

**Question:** Your application must force HTTPS and redirect all HTTP traffic to HTTPS without modifying application code. How would you configure this on ALB?

**Answer:**

* Attach an ACM-issued certificate to an HTTPS (443) listener and configure security policies with strong ciphers and TLS versions.
* Add an HTTP (80) listener with a rule that returns a 301/302 redirect to the HTTPS URL using host and path variables; no application change is required.
* Ensure the app is aware of secure context using headers like X-Forwarded-Proto so it generates correct callback URLs.
{% endstep %}

{% step %}
### Sticky sessions and scaling

**Question:** In one production app, enabling sticky sessions on ALB improved user experience but caused hotspot issues on some instances. How do you handle that?

**Answer:**

* Sticky sessions bind users to specific targets, creating uneven load. Monitor per-instance CPU and request counts to confirm hotspots.
* Limit stickiness duration so load can rebalance when instances are added/removed. For stateless services, prefer disabling stickiness and storing session state in Redis/ElastiCache or a database.
* If stickiness is required, over-provision capacity slightly and ensure ASG scales based on per-target metrics (e.g., ALB requests per target) to compensate.
{% endstep %}

{% step %}
### Cost optimization with multiple LBs

**Question:** Your AWS bill shows multiple ALBs across non-critical environments (dev, test, staging), each lightly used. How would you optimize cost without hurting agility?

**Answer:**

* Consolidate environments where possible: use one shared ALB per environment type or a single shared ALB with host- or path-based routing per project/environment (e.g., dev-app1.example.com, dev-app2.example.com).
* Use separate target groups and strict security boundaries (SGs, IAM, namespaces) to keep isolation logical while avoiding per-ALB hourly + LCU overhead.
* For internal, low-traffic services, consider a single internal ALB with multiple listeners/rules or evaluate whether legacy CLB suffices for some very simple cases (with caveats).
{% endstep %}

{% step %}
### Protecting from abusive clients

**Question:** An external client is flooding your public ALB with requests, impacting other users. What would you implement to protect the service?

**Answer:**

* Attach AWS WAF to the ALB and create rate-based rules to throttle IPs that exceed a safe requests-per-interval threshold.
* Use WAF rules for Geo/IP blocklists, bot control, and pattern matches against suspicious URIs or headers.
* Monitor ALB RequestCount and WAF metrics, and if abuse persists, consider putting CloudFront in front of ALB for extra shielding and caching.
{% endstep %}

{% step %}
### Access control with private NLB

**Question:** You have an internal microservice exposed via a private NLB that must only be reachable from specific VPCs. How would you enforce access control?

**Answer:**

* Use VPC peering or Transit Gateway and restrict inbound security groups or NACLs on the target subnets so only CIDRs or SGs from approved VPCs can reach the target ports.
* For NLB with IP targets, ensure only trusted source ranges are allowed; remember NLB itself is not a firewall, so primary control is SGs and routing.
* For cross-account access, expose the service via PrivateLink (NLB-backed endpoint service) and grant permissions only to specific AWS accounts/principals.
{% endstep %}

{% step %}
### Logging, durability, and analysis

**Question:** How do you design load balancer logging so that data is durable and easy to analyze in production?

**Answer:**

* Enable ALB/NLB access logs to an S3 bucket with lifecycle rules: keep detailed logs hot for a short period (e.g., 30–90 days) and transition older logs to infrequent access or Glacier to control cost while preserving history.
* Organize logs using a partitioned prefix (year/month/day/load-balancer-name) to make Athena or Glue queries efficient.
* Restrict S3 bucket access via IAM policies and optionally enable S3 Object Lock for compliance; integrate Athena, OpenSearch, or third-party tools for query and dashboarding.
{% endstep %}

{% step %}
### Cross-account ALB access (centralized ingress)

**Question:** Your organization uses a hub-and-spoke model with shared services in a central VPC. How would you expose services via ALB to workloads in other accounts without going over the public internet?

**Answer:**

* Put the ALB in the shared services VPC as internal; connect spoke VPCs via Transit Gateway or VPC peering so they can reach ALB subnets privately.
* Use Route 53 private hosted zones shared across accounts (or per-VPC associations) so services resolve to the internal ALB addresses.
* Enforce access via security groups referencing peered or TGW-attached VPCs, plus IAM and application-level auth; avoid public exposure unless necessary.
{% endstep %}

{% step %}
### Health checks pass, but app still broken

**Question:** You see all targets as healthy under ALB, but users report functional issues (incorrect responses, partial failures) for certain APIs. What does this indicate and how do you handle it?

**Answer:**

* Health checks often test a narrow path (e.g., /health); main app paths can still be broken while health checks remain green.
* Add more meaningful health checks that validate key dependencies (DB, cache, third-party APIs) and consider separate target groups with stricter checks for critical endpoints.
* Use ALB logs and application logs to correlate failing endpoints; introduce canary checks or synthetic monitoring for those specific URLs.
{% endstep %}

{% step %}
### Troubleshooting failed ELB health checks (legacy)

**Question:** On a legacy system using a Classic Load Balancer, instances frequently flap between healthy and unhealthy. How do you systematically debug this?

**Answer:**

* From the instance, run curl/telnet against the CLB health check port/path to confirm the service responds within the configured timeout; if slow, increase health check timeout/interval.
* Review instance logs and metrics (CPU, memory, GC, connection count) at times they go unhealthy; under heavy load, scale out or tune the service.
* Confirm the instance SG allows traffic from the ELB, and ensure no local firewall (iptables) blocks the health check port. Verify the health check port matches where the app listens.
{% endstep %}

{% step %}
### Migrating from CLB to ALB

**Question:** Your org wants to migrate a classic web app from Classic Load Balancer to ALB without major downtime. How would you plan it?

**Answer:**

* Create a new ALB in parallel with required listeners, certificates, security groups, and a new target group pointing to the same instances (or ASG) used by CLB.
* Update instance/app security groups to allow traffic from both CLB and ALB SGs, then test the ALB via a separate DNS name or host header to confirm routing and health.
* When validated, update DNS (Route 53) to point the main application hostname to ALB with a low TTL for quick rollback; keep CLB for a short overlap, then decommission to reduce cost.
{% endstep %}

{% step %}
### End-to-end TLS and compliance

**Question:** A compliance requirement states that traffic must be encrypted from client to ALB and from ALB to backend instances. How do you implement and maintain this?

**Answer:**

* Terminate TLS at ALB using ACM certificates for the public side and configure a secure TLS policy; configure the target group to use HTTPS to targets, with instances having server certificates (self-signed or private CA).
* Restrict instance SGs to allow only HTTPS from ALB SG; optionally validate backend certificates (hostname and CA) if you want full mutual trust.
* Automate certificate rotation on instances using SSM/ACM Private CA or your org’s PKI and validate in staging before enabling strict checks in production.
{% endstep %}

{% step %}
### Choosing between ALB and NLB for microservices

**Question:** You’re designing ingress for an EKS-based microservices platform. Some services are HTTP/REST, others are gRPC, and a few are plain TCP. How do you choose and combine ALB/NLB?

**Answer:**

* For HTTP/REST and gRPC over HTTP/2, use ALB with path- and host-based routing; it integrates well with EKS ingress controllers and supports advanced L7 features.
* For raw TCP/UDP services (e.g., custom protocols, databases, message brokers), use NLB for low latency and source IP preservation; EKS can expose them with type LoadBalancer and NLB annotations.
* Where appropriate, front ALB with CloudFront for caching and global edge presence, but keep NLB for pure transport-level services.
{% endstep %}

{% step %}
### Diagnosing “can’t connect to service” via NLB

**Question:** Applications in another VPC report “connection timed out” when trying to reach a service exposed via NLB. NLB shows targets as healthy. What are your deep-dive steps?

**Answer:**

* Verify routing: ensure VPC peering or Transit Gateway exists, correct route tables point to NLB subnets, and no overlapping CIDRs prevent routing.
* Check security groups and NACLs on both NLB target subnets and client subnets; confirm inbound/outbound rules allow the required ports and that NACLs are not dropping return traffic.
* Use VPC Flow Logs on both sides to see whether traffic reaches the NLB subnets and whether responses are sent back; flow logs reveal ACCEPT/REJECT patterns.
* If networking looks correct, validate the application by connecting from an instance in the same VPC as the NLB to rule out app-level binding or local firewall issues.
{% endstep %}
{% endstepper %}
