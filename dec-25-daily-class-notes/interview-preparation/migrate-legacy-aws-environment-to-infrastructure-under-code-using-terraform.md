# migrate legacy aws environment to infrastructure under code using terraform

In a previous project, the team inherited a large legacy AWS environment with manually provisioned EC2 instances, networking, and security configurations. The challenge was to bring the entire infrastructure under code using Terraform and automate deployments with zero downtime.

## What I did

* Analyzed the existing infrastructure and broke down resources into logical Terraform modules for VPC, subnets, security groups, EC2 instances, RDS, and load balancers.
* Developed reusable Terraform modules with variables and outputs to support different environments (dev, staging, prod) with isolated states using Terraform workspaces.
* Established remote S3 backend with DynamoDB for state locking to enable team collaboration.
* For application deployments on EC2 behind an Application Load Balancer (ALB), implemented blue-green deployment strategy using Terraform combined with Jenkins pipelines:
* Created two identical sets of instances (blue and green) registered with the ALB target groups.
* Jenkins pipeline took care of provisioning green instances, deploying the application, running smoke tests, and switching ALB target groups to green upon success.
* Rollback was as simple as switching back to blue target groups.
* Automated the full pipeline including Terraform apply, Ansible configuration management on instances, and post-deployment validation scripts in Jenkins.
* Added automated tagging, IAM role management, and security group auditing as part of modules for governance and compliance.

## Challenges & Solutions

* Handling resource dependencies was complex, especially with multi-tier apps relying on RDS connectivity and ALB target registration.
* I used explicit Terraform `depends_on` and output variables to manage resource creation order robustly.
* During initial deployments, some EC2 instances failed health checks due to startup script timing issues.
* Introduced retries and wait-for-health-check steps in Jenkins pipeline to improve reliability.
* Managed drift detection by implementing scheduled Terraform plan jobs to alert on manual infrastructure changes.
* Educated the team on Terraform best practices and modular design so the codebase remained maintainable and extensible.

## Results

* Reduced manual provisioning time from days to minutes.
* Achieved near-zero downtime deployments with blue-green strategy.
* Improved infrastructure consistency and version control, helping multiple teams collaborate efficiently.
* Pipeline automation enabled rapid environment provisioning for testing and reduced environment parity issues.

## Infrastructure Assessment & Modularization

We started with an extensive audit of the existing AWS infrastructure, which was mostly provisioned manually via the console and some legacy CloudFormation stacks. The environment had:

* Multiple VPCs with segmented subnets (public/private).
* EC2 instances running a multi-tier web application behind an ALB.
* RDS instances for database layers.
* Security groups with somewhat inconsistent rules.

I decomposed these into Terraform modules focusing on reusable components:

* Network module: managed VPC, subnets, route tables, internet gateways, NAT gateways.
* Security module: created reusable security group definitions with parameterized ingress and egress rules.
* Compute module: for EC2 instances with launch configurations and auto-scaling groups.
* Database module: for managing RDS instances with appropriate subnet groups and parameter groups.
* Load balancer module: ALB setup with listener rules and target groups for blue and green environments.

This modular design let us instantiate multiple environments consistently and supported Terraform workspaces to isolate state per environment.

## Blue-Green Deployment Pipeline

For application updates on EC2 instances behind the ALB, we devised a blue-green deployment strategy as follows:

{% stepper %}
{% step %}
### Terraform provisioning

Jenkins pipeline would trigger Terraform to provision the green environment (new EC2 instances in a separate ASG and target group). Terraform outputs included ALB target group ARNs and instance details.
{% endstep %}

{% step %}
### Configuration and deployment

After provisioning, Jenkins launched Ansible playbooks to install and configure the app on green instances.
{% endstep %}

{% step %}
### Smoke tests and traffic switch

Smoke tests and health checks ran automatically via Jenkins. If tests passed, Jenkins updated the ALB listener rules to redirect traffic from blue target group to green target group. DNS and routing stayed the same, ensuring zero downtime.
{% endstep %}

{% step %}
### Rollback

In case of failure, the pipeline automatically reverted the ALB traffic to blue instances. Green instances were then terminated to prevent cost leakage.
{% endstep %}
{% endstepper %}

## Technical Challenges

* Dependency management: We had to ensure RDS was fully available before EC2 instances started provisioning. We achieved this with explicit `depends_on` and checking DB availability in deployment scripts.
* Health check timing: EC2 instances sometimes appeared unhealthy initially due to app startup time. We added pre-checks with retries to wait for the app readiness before marking instances healthy.
* State management: As multiple teams worked concurrently, strict state locking with S3 and DynamoDB prevented race conditions.
* Secret management: Introduced AWS Secrets Manager integration with Ansible to securely inject DB credentials during configuration.

## CI/CD Pipeline Highlights

* Declarative Jenkinsfiles enabled environment parameterization (blue or green).
* Integrated Terraform init/plan/apply stages handled by Jenkins agents.
* Ansible playbooks triggered post-provisioning with dynamic inventory from Terraform outputs.
* Post-deployment validation included REST API health checks and database connectivity tests.
* Slack notifications and detailed logging improved deployment visibility.

## Outcome

This approach drastically improved deployment reliability and reduced manual errors. Infrastructure was now fully version-controlled, repeatable, and auditable.
