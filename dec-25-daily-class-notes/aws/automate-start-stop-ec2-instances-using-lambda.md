# automate start stop ec2 instances using lambda

![header image](../../.gitbook/assets/1nkY4YzPBymWnvig5fFnNwg.png)

In today’s fast-paced world of cloud computing, optimizing resources and minimizing costs are essential tasks. Amazon Web Services (AWS) offers tools like AWS Lambda to automate actions such as starting and stopping EC2 instances. This guide walks through automating the start-stop cycle of EC2 instances using Lambda and CloudWatch/EventBridge to reduce costs by running instances only when needed.

## Introduction to EC2 Instance Automation

Amazon Elastic Compute Cloud (EC2) provides scalable computing capacity in the AWS cloud. It’s common to have instances running 24/7 even when not in use. Automating start/stop with Lambda ensures instances run only when needed, optimizing expenses.

## Prerequisites

Make sure you have:

* An AWS account with appropriate permissions.
* EC2 instances you want to automate.
* Basic knowledge of AWS services like EC2, IAM, Lambda, and CloudWatch.

![prereq image](<../../.gitbook/assets/1iBdp VWpwm 3EZOmfusJ4A.png>)

## Setting up IAM Role and Policy

The Lambda function needs an IAM role that allows reading/listing EC2 instances and describing EC2 tags. Create the required policy and role in the IAM console.

{% stepper %}
{% step %}
### Create the IAM policy

* Go to the AWS Management Console → IAM dashboard → Policies → Create policy.
* Configure the policy to allow at least the following EC2 actions: DescribeInstances, StartInstances, StopInstances (or attach a managed policy such as `AmazonEC2FullAccess` if appropriate for your needs).
* Save the policy.

![create policy](../../.gitbook/assets/1_G93ithegduPDk5qZn8zuQ.png)
{% endstep %}

{% step %}
### Create the IAM role for Lambda

* In the IAM dashboard, click Roles → Create role.
* Choose "Lambda" as the trusted entity type.
* Attach policies such as `AmazonEC2FullAccess` and `CloudWatchEventsFullAccess` (or the custom policy you created).
* Name the role and create it.

![create role](../../.gitbook/assets/1v7i6WIadCwwvaKjtJonORw.png) ![select lambda](<../../.gitbook/assets/1MYKzPcsOMnPu D5bNkdcAg.png>)
{% endstep %}
{% endstepper %}

![iam screenshots](../../.gitbook/assets/10p0Ovj0G8t5rXrfdeUQLWQ.png) ![iam screenshots 2](../../.gitbook/assets/1cfrsIMuoDdABmyJCwY0PAw.png) ![iam screenshots 3](../../.gitbook/assets/1XNPtUCWjPz9FmszvTrxOag.png)

## Creating a Lambda Function

Create two Lambda functions (one to stop instances and one to start instances). Use Python 3.9 runtime and the IAM role you created.

{% stepper %}
{% step %}
### Create the "Stop EC2 Instances" Lambda

1. Go to AWS Services → Lambda → Create function.
2. Choose "Author from scratch".
3. For Function name, use something like `StopEC2Instances`.
4. For Runtime, choose Python 3.9.
5. Under Permissions → Change default execution role → Use an existing role.
6. Choose the IAM role you created.
7. Create function.
8. In the Code section, paste the stop function code (see below).
9. Choose Deploy.

![lambda create](<../../.gitbook/assets/1QAL9w0n5jiBuMyQQZ WogQ.png>)
{% endstep %}

{% step %}
### Create the "Start EC2 Instances" Lambda

Repeat the steps above to create a second function (for example, `StartEC2Instances`) and paste the start function code (see below). Deploy the function.

![lambda list](../../.gitbook/assets/1bfQK2vixOp9syiEcyKjS8g.png)
{% endstep %}
{% endstepper %}

### Lambda code examples

Stop instances:

{% code title="stop_instances.py" %}
```python
# Lambda code reference to stop EC2

import boto3
region = 'us-west-1'
instances = ['i-12345cb6de4f78g9h', 'i-08ce9b2d7eccf6d26']
ec2 = boto3.client('ec2', region_name=region)

def lambda_handler(event, context):
    ec2.stop_instances(InstanceIds=instances)
    print('stopped your instances: ' + str(instances))
```
{% endcode %}

Start instances:

{% code title="start_instances.py" %}
```python
# Lambda code reference to start EC2

import boto3
region = 'us-west-1'
instances = ['i-12345cb6de4f78g9h', 'i-08ce9b2d7eccf6d26']
ec2 = boto3.client('ec2', region_name=region)

def lambda_handler(event, context):
    ec2.start_instances(InstanceIds=instances)
    print('started your instances: ' + str(instances))
```
{% endcode %}

Replace the instance IDs in the `instances` list with your actual instance IDs.

![lambda console](<../../.gitbook/assets/11ONAd zdNM6COhOe ssVIA.png>)

## Setting Up CloudWatch Events (EventBridge Scheduler)

Use CloudWatch Events (EventBridge) to schedule the Lambda functions.

{% stepper %}
{% step %}
### Create a rule to stop instances on schedule

1. Go to the AWS Management Console → CloudWatch (or EventBridge).
2. In the dashboard, go to Rules (under Events) → Create rule.
3. Enter a Name, e.g., `StopEC2Instances`, and optionally a Description.
4. For Rule type, choose Schedule → Continue in EventBridge Scheduler.
5. For Schedule pattern, choose Recurring schedule.
6. Choose Schedule type:
   * For a rate-based schedule, enter a rate expression (e.g., every X hours).
   * For a cron-based schedule, enter a Cron expression.
7. In Select targets, choose Lambda function as the Target and select your stop function.
8. Review and Create.

Note: Cron expressions are evaluated in UTC. Adjust for your timezone.
{% endstep %}

{% step %}
### Create a rule to start instances on schedule

Repeat the steps to create another rule named, for example, `StartEC2Instances`, and select your start Lambda as the target. Configure the schedule (rate or cron) to match when you want the instances started.
{% endstep %}
{% endstepper %}

![eventbridge screenshot 1](../../.gitbook/assets/1mY2_DiZhaoASRKknOuyqaQ.png) ![eventbridge screenshot 2](../../.gitbook/assets/1xk2oNdpiu75zxwuWLKh3Mg.png) ![eventbridge screenshot 3](../../.gitbook/assets/14qDYPwmu8hc9VAagEc_7ew.png) ![eventbridge screenshot 4](../../.gitbook/assets/1MSt3Vr6uS1SLyA0ezIZ_zw.png)

{% hint style="info" %}
For details on cron expression syntax for EventBridge/CloudWatch schedule expressions, see: https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/ScheduledEvents.html Also remember: Cron expressions are evaluated in UTC.
{% endhint %}

## Testing the Automation

After creating rules, the Lambda functions will be triggered according to the schedules and will start/stop the specified instances. Monitor CloudWatch Logs and the EC2 console to verify behavior.

* Check CloudWatch Logs for Lambda execution logs.
* Verify instance states in the EC2 console.

This completes the setup for automating EC2 start/stop cycles using AWS Lambda and scheduled CloudWatch/EventBridge rules.
