# AWS Topics

{% stepper %}
{% step %}
### IAM – identity & access

* “AWS Identity and Access Management (IAM) lets you securely control who can access which AWS resources, using users, groups, roles, and policies.”
* “Best practice is: never use the root account; instead create IAM users and roles with least‑privilege policies (JSON permissions documents) and use them via console, CLI, or SDK.”
* “In real projects, EC2, Lambda, and other services assume IAM roles so they can talk to S3, RDS, or other services without hard‑coded credentials.”

Trainer actions / demo idea:

* Show IAM user vs role, and attach a simple S3 read‑only policy to a role.
* Emphasize least‑privilege and temporary credentials (roles, STS).
{% endstep %}

{% step %}
### VPC – your private network

* “Amazon VPC (Virtual Private Cloud) is a logically isolated virtual network where you launch your AWS resources, with your own IP ranges, subnets, route tables, and gateways.”
* “Typical pattern: public subnets for internet‑facing components (ALB, bastion, NAT gateway) and private subnets for app servers and databases, controlled using security groups and network ACLs.”
* “Placing RDS and internal services only in private subnets is a key security best practice; internet traffic hits the web/app layer but never talks to the database directly.”

Trainer actions / whiteboard:

* Draw a simple VPC with 2 public and 2 private subnets, an EC2 instance in public, RDS in private, and S3 outside VPC.
* Highlight security groups vs network ACLs and the role of NAT gateway for outbound traffic from private subnets.
{% endstep %}

{% step %}
### EC2, S3, RDS, Lambda – core services

#### EC2 – virtual servers

* “Amazon EC2 (Elastic Compute Cloud) provides resizable virtual servers (instances) in the cloud for running applications, APIs, and background jobs.”
* “You choose instance type (CPU/RAM), OS AMI, networking (subnet, security groups), storage (EBS), and can auto scale up or down based on demand to control performance and cost.”

#### S3 – object storage

* “Amazon S3 (Simple Storage Service) is highly durable object storage for files like logs, backups, images, and static website assets, accessible over HTTP.”
* “It uses buckets and objects, supports versioning, lifecycle rules, cross‑region replication, and fine‑grained IAM‑based access control, with pay‑as‑you‑go pricing.”

#### RDS – managed relational DB

* “Amazon RDS (Relational Database Service) is a managed database service supporting engines like MySQL, PostgreSQL, and others, handling backups, patching, and failover.”
* “You typically deploy RDS inside a VPC in private subnets, use security groups for access control, and scale vertically (bigger instance) or via read replicas for heavy read traffic.”

#### Lambda – serverless compute

* “AWS Lambda lets you run code without managing servers, triggered by events such as API Gateway calls, S3 uploads, or CloudWatch schedules.”
* “You pay only for execution time, which is very useful for glue logic, micro‑APIs, cron‑like jobs, and event‑driven architectures.”

Trainer tips:

* For each service, include a 3–5 minute demo or screenshot walkthrough (launch EC2, create S3 bucket, show RDS console, inspect a Lambda function trigger).
{% endstep %}

{% step %}
### Putting it together: 2‑tier app

Explain and sketch this architecture:

* “A typical 2‑tier web application on AWS uses a VPC for isolation, EC2 instances in public or private subnets for the web/app tier, RDS in private subnets for persistence, and S3 for static assets or backups.”
* “Traffic flows: user → internet → load balancer/EC2 in public subnet → RDS in private subnet, while logs/backups or static content go to S3; IAM roles and security groups secure each hop.”

DevOps closure:

* “On top of this, DevOps teams add CodeCommit/CodeBuild/CodeDeploy/CodePipeline to automate builds, tests, and deployments of the application into EC2 or Lambda, enabling continuous delivery on AWS.”
* “Monitoring and logging are usually handled using CloudWatch and related services, so that the pipeline plus runtime environment are both observable and auditable.”

Trainer actions:

* Sketch the flow while describing each component’s role.
* Optionally run a short demo showing deployment pipeline artifacts or a CloudWatch dashboard.

If you tell me the class duration (e.g., 60 or 90 minutes), I can provide a minute‑by‑minute plan with activities and questions.
{% endstep %}
{% endstepper %}

***

References

⁂
