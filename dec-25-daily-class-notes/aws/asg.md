# asg

### Brief Introduction to AWS ASG

Start the session by introducing the concept of Auto Scaling Groups (ASG):

* Definition: An ASG in AWS automatically manages the number of EC2 instances according to demand, enabling applications to scale in and out based on set policies.
* Components:
  * Launch Template/Configuration: Blueprint for EC2 instances (AMI, type, security groups, etc.)
  * Scaling Policies: Rules for adding/removing instances (target tracking, step scaling, schedule-based)
  * Health Checks: Mechanism to replace unhealthy instances
  * Load Balancer (optional): Evenly distributes traffic among instances
  * Desired, Minimum, Maximum Capacity: Controls the flexibility of scaling

### Real-Time Practical Scenario

Scenario: “You’re running a web application that experiences increased traffic during business hours and reduced load during nights and weekends.”

### Hands-on Lab Steps

{% stepper %}
{% step %}
### Set up a Launch Template

* Choose an Amazon Machine Image (AMI) (e.g., Amazon Linux).
* Define instance type (t2.micro for demo).
* Attach key pair for SSH access.
{% endstep %}

{% step %}
### Create a Target Group and an Application Load Balancer (ALB)

* Target group will receive instances launched by ASG.
* ALB will route incoming HTTP/HTTPS traffic to those instances.
{% endstep %}

{% step %}
### Create an Auto Scaling Group

* Associate with launch template and target group.
* Set capacities:
  * Minimum: 1
  * Maximum: 4
  * Desired capacity: 2
{% endstep %}

{% step %}
### Configure Scaling Policies

* Target Tracking: Maintain average CPU at 50%.
* Step Scaling: Add 1 instance if CPU > 70% for 5 minutes, remove 1 if < 30%.
{% endstep %}

{% step %}
### Simulate Traffic and Scaling

* Use a tool like Apache Benchmark or scripts to generate load.
* Observe instance scale-out when CPU spikes; scale-in when load drops.
{% endstep %}

{% step %}
### Demonstrate Fault Tolerance

* Manually stop an EC2 instance from the ASG and show that ASG will replace it to maintain desired capacity.
{% endstep %}

{% step %}
### Bonus: Scheduled Scaling

* Set up scheduled scaling to increase capacity during specific hours (e.g., scale to 4 from 9 AM - 6 PM, and down to 1 otherwise).
{% endstep %}
{% endstepper %}

### Real-World Examples

* E-Commerce Sites: Scale up for seasonal sales and scale down post-sale to save cost.
* News Portals: Quickly handle surges during breaking news.
* SaaS Applications: Match user demand across global time zones.

{% hint style="info" %}
Best Practices to Share

* Always configure health checks (EC2 and ELB) for accurate recovery.
* Use launch templates for consistency and versioning.
* Test scaling policies periodically.
* Enable notifications (SNS) for scaling events.
{% endhint %}

## Capacity

* Desired Capacity: The ideal number of EC2 instances you want running in your ASG at any given time. The ASG tries to maintain this number under normal conditions. When scaling activities are triggered (like a scheduled change or CloudWatch alarm), the desired capacity is adjusted up or down.
* Minimum Capacity: The lowest number of EC2 instances that your ASG will maintain. The ASG will not terminate instances below this number.
* Maximum Capacity: The highest number of EC2 instances that your ASG can launch. No scaling policies or manual actions will increase the group beyond this limit.

Example:

* Minimum: 2
* Desired: 3
* Maximum: 5

Your ASG will launch 3 instances by default, never let the count fall below 2, and never let it increase above 5—no matter what scaling event occurs.

Additional concepts:

* Warmup/Cooldown: Settings to manage how quickly scaling actions repeat.
* Health Checks: ASG replaces unreachable/unhealthy instances automatically.
* Scaling Limits: Set limits on how much your desired capacity can be increased or decreased. The minimum must be ≤ desired, and maximum must be ≥ desired.

## Scaling Modes

* Manual scaling: Change desired capacity yourself. ASG will not automatically adjust to workload.
* Automatic scaling: ASG adjusts instances in response to CloudWatch metrics or scheduled actions.

Automatic scaling policy options:

* No scaling policies: ASG remains static at its initial size.
* Target tracking scaling policy: Select a CloudWatch metric and target value (e.g., keep average CPU at 50%). ASG adjusts desired capacity to keep the metric near the target.
* Step scaling: Define threshold-based actions to respond more aggressively to big metric changes.
* Scheduled scaling: Change desired capacity at specific times (e.g., increase at 9 AM, decrease at 9 PM).

## Scenario-Based Questions for Students

<details>

<summary>1. You have a web app that peaks between 6–10 PM daily. How would you configure an ASG to save costs outside peak hours?</summary>

Discuss options such as scheduled scaling to lower desired capacity outside peak hours.

</details>

<details>

<summary>2. An ASG is scaling out, but traffic is not evenly distributed. What could be wrong?</summary>

Consider load balancer configuration, target group health, or session affinity/misrouting.

</details>

<details>

<summary>3. If the health check type is set to EC2, but the underlying web service fails, will ASG replace the instance? Why or why not?</summary>

EC2 health checks only detect instance-level reachability; they won't detect application-level failures unless ELB/health check integration is used.

</details>

<details>

<summary>4. Your ASG doesn’t scale in after load decreases. What checks would you perform?</summary>

Check cooldown/warmup settings, CloudWatch alarms, scaling policies, and activity history for pending terminations.

</details>

<details>

<summary>5. How would you integrate a custom alarm (e.g., queue backlog) as a scaling trigger for an ASG?</summary>

Create a CloudWatch metric or custom metric for the queue backlog and attach a scaling policy or alarm based on that metric.

</details>

<details>

<summary>6. Describe how you’d recover quickly if AWS announces scheduled maintenance on your availability zone.</summary>

Consider cross-AZ deployment, increasing capacity in other AZs, and replacing instances to maintain desired capacity in healthy AZs.

</details>
