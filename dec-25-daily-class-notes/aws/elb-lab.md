# elb lab

{% stepper %}
{% step %}
### Prerequisites and Setup

* **AWS Account & Permissions**
  * Ensure each student has an AWS account with “AdministratorAccess” or equivalent privileges.
* **VPC and Subnet**
  * Use the default VPC in a single AWS Region (e.g., us-east-1).
  * Confirm at least two public subnets exist.
* **Key Pair**
  * Create or select an existing EC2 key pair for SSH access.
{% endstep %}

{% step %}
### EC2 Instance Deployment

* **Launch Two EC2 Instances**
  * AMI: Amazon Linux 2
  * Instance Type: t2.micro (Free Tier)
  * Network: Default VPC, two different public subnets
  * Security Group “web-sg”: allow inbound TCP 80 (HTTP) and 22 (SSH) from your IP
  * Key Pair: your chosen key
* **Install Web Server**
  *   SSH into each instance:

      <pre class="language-bash" data-title="SSH into instance"><code class="lang-bash">ssh -i "your-key.pem" ec2-user@&#x3C;EC2_Public_IP>
      </code></pre>
  *   Install Apache:

      <pre class="language-bash" data-title="Install Apache"><code class="lang-bash">sudo yum update -y
      sudo yum install httpd -y
      sudo systemctl start httpd
      sudo systemctl enable httpd
      </code></pre>
  *   Create distinct homepages:

      <pre class="language-bash" data-title="Create distinct homepages"><code class="lang-bash">echo "Welcome to Server 1" | sudo tee /var/www/html/index.html   # on instance #1
      echo "Welcome to Server 2" | sudo tee /var/www/html/index.html   # on instance #2
      </code></pre>
{% endstep %}

{% step %}
### Configure Health Checks

* **Modify Security Group for Health Checks**
  * In “web-sg”, ensure inbound TCP 80 from the ALB’s security group (to be created) is allowed.
* **Verify Web Servers**
  * In a browser, visit each EC2’s public IP; confirm distinct messages.
{% endstep %}

{% step %}
### Create Application Load Balancer

1. Navigate to EC2 > Load Balancers > Create Load Balancer
2. Select “Application Load Balancer”
   * Name: `lab-alb`
   * Scheme: internet-facing
   * Listeners: HTTP:80
   * Availability Zones: Select both subnets you used for EC2 instances
3. Configure Security Group for ALB
   * Create “alb-sg”: allow inbound TCP 80 from 0.0.0.0/0
4. Configure Target Group
   * Target group name: `lab-tg`
   * Target type: instance
   * Protocol: HTTP, Port: 80
   * Health check path: `/`
5. Register Targets
   * Add both EC2 instances to `lab-tg`
6. Review & Create
   * Confirm settings and click “Create load balancer”
{% endstep %}

{% step %}
### Test Load Balancer Behavior

* **Obtain ALB DNS Name**
  * In Load Balancers list, copy the DNS (e.g., `lab-alb-123456.elb.amazonaws.com`).
* **Validate Round-Robin**
  * In a browser, visit the DNS URL repeatedly; pages should alternate between “Server 1” and “Server 2.”
* **Simulate Failure**
  *   SSH into one EC2 instance and stop Apache:

      <pre class="language-bash" data-title="Stop Apache"><code class="lang-bash">sudo systemctl stop httpd
      </code></pre>
  * Refresh the ALB URL; only the healthy instance’s page appears.
  *   Restart Apache on stopped instance:

      <pre class="language-bash" data-title="Start Apache"><code class="lang-bash">sudo systemctl start httpd
      </code></pre>
{% endstep %}

{% step %}
### Optional Enhancements

* **Enable Sticky Sessions**
  * Edit `lab-tg` attributes → Enable “Stickiness” (duration e.g. 300 seconds)
* **Path-Based Routing**
  * In ALB listener rules, add rule:
    * If path is `/app1*`, forward to a second target group (you may launch additional instances)
* **Auto Scaling**
  * Create an Auto Scaling group behind `lab-tg` with min=2, max=4, scale-out on CPU > 50%
{% endstep %}

{% step %}
### Cleanup

* Terminate EC2 instances
* Delete Load Balancer, Target Group, Security Groups
{% endstep %}
{% endstepper %}

### Scenario-Based Questions (FAQ)

<details>

<summary>1. If one EC2 instance fails its health check, how does the ALB respond?</summary>

The ALB stops routing traffic to that unhealthy target. It continues sending requests only to healthy targets in the target group until the failed instance passes health checks again.

</details>

<details>

<summary>2. After adding two more instances, what steps register them with the ALB?</summary>

Register the new instances with the existing target group (`lab-tg`)—select the target group, choose “Register targets,” then add the instances (or their instance IDs) and ensure they pass health checks.

</details>

<details>

<summary>3. How would you restrict direct traffic to EC2, allowing only the ALB to reach them?</summary>

Modify the EC2 instances' security group (`web-sg`) to allow inbound HTTP (port 80) only from the ALB security group (`alb-sg`) rather than 0.0.0.0/0.

</details>

<details>

<summary>4. If the health-check path returns HTTP 500, what happens to that target?</summary>

The ALB marks the target as unhealthy after the configured unhealthy threshold is met and stops routing traffic to it until health checks succeed.

</details>

<details>

<summary>5. Design a multi-region ALB architecture for cross-region redundancy.</summary>

Use region-level ALBs in each region fronting regional target groups/ASGs. Place a global routing layer (Route 53 latency-based or failover routing, or AWS Global Accelerator) to direct traffic to the healthiest/closest region.

</details>

<details>

<summary>6. When are sticky sessions useful, and what drawbacks do they introduce?</summary>

Sticky sessions are useful when session state is stored locally on an instance and you need a client to keep hitting the same backend. Drawbacks: uneven load distribution, reduced fault tolerance, and scaling complications; better to use stateless apps or centralized session stores.

</details>

<details>

<summary>7. Which AWS metrics would you monitor to detect ALB performance issues?</summary>

Monitor metrics such as:

* RequestCount
* TargetResponseTime
* HealthyHostCount / UnHealthyHostCount
* HTTP 4xx/5xx counts
* ActiveConnectionCount / NewConnectionCount

</details>
