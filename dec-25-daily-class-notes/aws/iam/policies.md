# policies

types of policies in aws types of policies in aws

### 1. Identity-Based Policies

* **Managed Policies**: Standalone policies you can attach to multiple identities (users, groups, or roles).
  * _AWS Managed Policies_: Created and maintained by AWS, covering common use cases.
  * _Customer Managed Policies_: Created and maintained by you, providing custom permissions.
* **Inline Policies**: Directly embedded within a single user, group, or role. These have a one-to-one relationship with the identity and aren’t reusable.

### 2. Resource-Based Policies

* Attached directly to AWS resources (e.g., S3 buckets, SNS topics, SQS queues).
* Define who (principals) can access the resource and what actions they may perform.
* Useful for cross-account access control

### 3. Access Control Lists (ACLs)

* Attached to certain resources, such as S3 buckets, mainly for legacy and cross-account access management.
* ACLs do not use the JSON policy document structure and cannot grant permissions to entities within the same account.

### Quick Reference Table

| Policy Type             | Main Purpose                                 | Where Used                      |
| ----------------------- | -------------------------------------------- | ------------------------------- |
| Managed (AWS/Customer)  | Standard/repeatable identity permissions     | Users, groups, roles            |
| Inline Policies         | Customized, entity-specific permissions      | Individual users, groups, roles |
| Resource-Based Policies | Access control granted directly on resources | Resources (S3, SNS, SQS, etc.)  |
| Access Control Lists    | Legacy/cross-account permissions             | S3, others (resource level)     |

These policy types together allow you to apply _precise permission management_ suited to a variety of AWS organizational structures and security requirements.

***

To create an **inline policy** in AWS (embedded directly within a single user, group, or role), you can use the AWS Management Console, AWS CLI, or AWS API. Below are guides for the AWS Management Console and the CLI.

### Creating an Inline Policy Using the AWS Management Console

{% stepper %}
{% step %}
### Open IAM and choose identity

* Sign in to the AWS Management Console and open the IAM console.
* In the navigation pane, choose either **Users**, **User groups**, or **Roles** (depending on where you want to add the inline policy).
* Select the specific user, group, or role from the list.
* Go to the **Permissions** tab.
* Click **Add permissions**, then choose **Create inline policy**.
{% endstep %}

{% step %}
### Create the policy

You’ll see two options to define the permissions:

* **Visual editor:** Build your policy step by step by selecting services, actions, and resources (great for beginners—no need to write JSON).
* **JSON:** Paste or type your custom JSON policy document (ideal for advanced users or to copy-paste sample policies).
{% endstep %}

{% step %}
### Review and create

* After defining the permissions, review the policy.
* Give your inline policy a **name**.
* Click **Create policy**.

Notes:

* The policy now applies only to the selected user, group, or role and cannot be reused elsewhere.
* If the identity (user, group, or role) is deleted, the inline policy is deleted too.
{% endstep %}
{% endstepper %}

### Example Inline Policy (JSON)

{% code title="inline-policy.json" %}
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListAllMyBuckets",
      "Resource": "*"
    }
  ]
}
```
{% endcode %}

### Creating Inline Policies Using AWS CLI

You can also use the CLI to add an inline policy (example for a user):

{% code title="AWS CLI" %}
```bash
aws iam put-user-policy \
  --user-name MyUser \
  --policy-name MyInlinePolicy \
  --policy-document file://policy.json
```
{% endcode %}

Replace `MyUser` with your IAM username and provide your policy JSON in `policy.json`.

{% hint style="info" %}
Key Points:

* Inline policies are unique to a single identity.
* They’re useful for highly specific, non-reusable permissions.
* Deleting the identity also deletes the inline policy automatically.
{% endhint %}

For more advanced scenarios, you can use conditions and specify resource ARNs just as you would in any IAM policy.

***

Sample scenario

An application or EC2 instance temporarily needing access to S3

To give an application or EC2 instance **temporary access to S3**, the recommended and secure AWS-native methods are described below.

### 1. Assign an IAM Role to the EC2 Instance

* **Best Practice:** Attach an IAM role with the minimum S3 permissions your application needs directly to the EC2 instance when launching it, or attach/change the role later.
* **How It Works:** The EC2 instance receives temporary security credentials (via the Instance Metadata Service), which the AWS SDKs and CLI use automatically—the credentials rotate every few hours.
* **Benefits:** No embedded long-term credentials in your code or instance, automatic credential rotation, and least-privilege security.
* **When Access Should Be Temporary:**
  * Remove or change the IAM role when access is no longer needed.
  * You can further restrict the role with a policy containing a time-based condition (e.g., using `aws:CurrentTime`) for strictly time-bounded access.

### 2. Use Pre-signed S3 URLs

* **For Temporary Object Access:** Generate a pre-signed URL for an object, which allows anyone with the link to access the S3 object _just for a limited time_ (you specify the expiry when generating).
* **Use Case:** Granting access to a specific object (e.g., downloading/uploading a file), not for listing or managing all S3 buckets/objects.
* **Generate via:** AWS CLI, SDKs, or Console actions.

### 3. Use AWS STS Temporary Credentials

* **Advanced:** Have your app or user assume a role with S3 permissions via the AWS Security Token Service (STS), receiving temporary credentials specifically for a session or workflow.
* **Use Case:** Federated/third-party users who should get temporary (session-limited) S3 access without attaching a permanent role to the EC2 instance.

### Example: Grant EC2 Instance Temporary S3 Access via IAM Role

{% stepper %}
{% step %}
### Create role with scoped S3 policy

Create an IAM role with an S3-access policy scoped to only the resources/actions you need.
{% endstep %}

{% step %}
### Attach role to EC2 instance

Attach the role to your EC2 instance (either at launch or by updating the instance’s IAM role).
{% endstep %}

{% step %}
### Remove or restrict after task

Remove the role after the task completes, or further restrict access by using IAM policy conditions to auto-expire authorization (e.g., with `aws:CurrentTime`).
{% endstep %}
{% endstepper %}

### Example: Grant Temporary S3 Object Access via Pre-Signed URL

* In the AWS Console, select your S3 object → "Create pre-signed URL" → choose expiry → share resulting URL. Access automatically expires.

{% hint style="warning" %}
Security & Best Practices

* Prefer temporary credentials (roles, STS, pre-signed URLs) over hard-coded keys.
* Limit scope (buckets, actions) and duration as much as possible.
* Remove or modify access immediately after your application or instance no longer needs it.
{% endhint %}

Summary Table

| Solution          | Typical Use Case                  | Expiry/Revocation Method         |
| ----------------- | --------------------------------- | -------------------------------- |
| IAM Role (on EC2) | App on EC2 needs S3 access        | Change/remove role, policy timer |
| Pre-signed S3 URL | Download/upload single S3 object  | Link expires (short time)        |
| STS AssumeRole    | Federated or session-based access | Token/session auto-expires       |

These methods ensure minimum, time-bounded access and follow AWS security best practices.
