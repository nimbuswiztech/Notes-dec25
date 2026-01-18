# Auto scaling groups

### Brief Introduction to AWS ASG

Start the session by introducing the concept of **Auto Scaling Groups**:

* **Definition:** An ASG in AWS automatically manages the number of EC2 instances according to demand, enabling applications to scale in and out based on set policies.
* **Components:**
  * **Launch Template/Configuration:** Blueprint for EC2 instances (AMI, type, security groups, etc.)
  * **Scaling Policies:** Rules for adding/removing instances (target tracking, step scaling, schedule-based)
  * **Health Checks:** Mechanism to replace unhealthy instances.
  * **Load Balancer (optional):** Evenly distributes traffic among instances.
  * **Desired, Minimum, Maximum Capacity:** Controls the flexibility of scaling.

### Real-Time Practical Scenario

**Scenario:** “You’re running a web application that experiences increased traffic during business hours and reduced load during nights and weekends.”

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
* Set:
  * Minimum: 1
  * Maximum: 4
  * Desired capacity: 2
{% endstep %}

{% step %}
### Configure Scaling Policies

* **Target Tracking:** Maintain average CPU at 50%.
* **Step Scaling:** Add 1 instance if CPU >70% for 5 minutes, remove 1 if <30%.
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

* **E-Commerce Sites:** Scale up for seasonal sales and scale down post-sale to save cost.
* **News Portals:** Quickly handle surges during breaking news.
* **SaaS Applications:** Match user demand across global time zones.

{% hint style="info" %}
Best Practices to Share

* Always configure health checks (EC2 and ELB) for accurate recovery.
* Use launch templates for consistency and versioning.
* Test scaling policies periodically.
* Enable notifications (SNS) for scaling events.
{% endhint %}

## Capacity

* **Desired Capacity:** The ideal number of EC2 instances you want running in your ASG at any given time. The ASG tries to maintain this number under normal conditions. When scaling activities are triggered (like a scheduled change or CloudWatch alarm), the desired capacity is adjusted up or down.
* **Minimum Capacity:** The lowest number of EC2 instances that your ASG will maintain. Even if there's no demand, the ASG will not terminate instances below this number.
* **Maximum Capacity:** The highest number of EC2 instances that your ASG can launch. No scaling policies or manual actions will increase the group beyond this limit.

Example:

* Minimum: 2
* Desired: 3
* Maximum: 5

Your ASG will launch 3 instances by default, never letting the count fall below 2, and never letting it increase above 5—no matter what scaling event occurs.

### Manual vs. Automatic Scaling

* **Manual scaling:** You adjust the number of instances yourself (change the desired capacity up or down). The ASG won’t change automatically based on workload demand.
* **Automatic scaling:** The ASG adjusts the number of instances in response to monitored metrics or schedules, keeping your application responsive and cost-efficient.

### Scaling Limits: Minimum, Desired, Maximum Capacities

* The **minimum desired capacity** must be equal to or less than the current desired capacity.
* The **maximum desired capacity** must be equal to or greater than the current desired capacity.
* The desired capacity stays within the min-max range.

### Key Concepts

* **Desired Capacity:** Number of instances ASG tries to maintain.
* **Min/Max Size:** Limits for instance count.
* **Warmup/Cooldown:** Settings to manage how quickly scaling actions repeat.
* **Health Checks:** ASG replaces unreachable/unhealthy instances automatically.

### Example Use of Policies

* Use _target tracking_ to maintain 50% CPU normally.
* Add a _step scaling_ policy to aggressively add instances if CPU exceeds 80% for quicker scaling during traffic spikes.

### Automatic Scaling Policies

* **No scaling policies:** The ASG remains static at its initial size.
* **Target Tracking Scaling Policy:** Select a CloudWatch metric (e.g., average CPU utilization) and set a target value (e.g., 50%). The ASG will attempt to keep the metric near that target.
* **Other scaling options** (configured after creation):
  * **Step scaling:** Responds to metric thresholds with defined actions.
  * **Scheduled scaling:** Changes desired capacity at set times (e.g., scale up at 9am, down at 9pm).

## Scenario-Based Questions for Students

<details>

<summary>1. You have a web app that peaks between 6–10 PM daily. How would you configure an ASG to save costs outside peak hours?</summary>

Consider using scheduled scaling to increase desired capacity during 6–10 PM and reduce it outside those hours. Combine with target tracking or step scaling for unexpected spikes during peak window.

</details>

<details>

<summary>2. An ASG is scaling out, but traffic is not evenly distributed. What could be wrong?</summary>

Possible causes include misconfigured load balancer target group, health check mismatches, or instances not registered with the load balancer. Check ALB/target group configuration and instance registration.

</details>

<details>

<summary>3. If the health check type is set to EC2, but the underlying web service fails, will ASG replace the instance? Why or why not?</summary>

If health checks are EC2-only, ASG will only detect instance-level failures (e.g., unreachable instance). It may not detect application-level failures. Use ELB/ALB health checks to detect web service failures so ASG can replace instances serving unhealthy application endpoints.

</details>

<details>

<summary>4. Your ASG doesn’t scale in after load decreases. What checks would you perform?</summary>

Check scaling policies, CloudWatch alarms, cooldown/warmup periods, and whether min capacity prevents scaling in. Verify metrics and that alarms have returned to non-breach state.

</details>

<details>

<summary>5. How would you integrate a custom alarm (e.g., queue backlog) as a scaling trigger for an ASG?</summary>

Create a CloudWatch metric or custom metric representing the queue backlog, then create an alarm on that metric and attach a scaling policy (target tracking or step scaling) to the ASG triggered by the alarm.

</details>

<details>

<summary>6. Describe how you’d recover quickly if AWS announces scheduled maintenance on your availability zone.</summary>

Options include reducing reliance on a single AZ by spreading ASG across multiple AZs, enabling cross-zone load balancing, or temporarily increasing capacity in other AZs. Also consider using scheduled scaling to shift capacity ahead of maintenance windows.

</details>
