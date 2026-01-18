# Introduction to AWS IAM

### 1. Introduction to AWS IAM

Start with a concise overview:

* **Definition:** IAM enables you to securely control access to AWS services and resources.
* **Purpose:** It helps manage users, groups, roles, and permissions in your AWS environment.
* **Use Case:** Multi-user AWS environments (e.g., a company with developers, admins, and auditors).

### 2. Key IAM Components with Examples

Explain each main component with practical, relatable analogies:

| IAM Component | Description                                          | Example/Analogy                                                   |
| ------------- | ---------------------------------------------------- | ----------------------------------------------------------------- |
| **Users**     | Individual with credentials to access AWS resources. | _A software developer in your company_                            |
| **Groups**    | Collection of users sharing permissions.             | _All developers placed in a “Developers” group_                   |
| **Roles**     | AWS identity with permissions, assumed temporarily.  | _An application or EC2 instance temporarily needing access to S3_ |
| **Policies**  | JSON documents that define permissions.              | _A policy allowing only S3 Read access_                           |

***

Exercise 1: Creating an IAM User

{% stepper %}
{% step %}
### Create the user

* Open the IAM service on AWS Console.
* Create a new user (e.g., “student1”).
{% endstep %}

{% step %}
### Configure access

* Assign console access and set a custom password.
{% endstep %}

{% step %}
### Attach permissions

* Attach an existing policy (e.g., `AmazonS3ReadOnlyAccess`).
{% endstep %}

{% step %}
### Verify

* Log in as the new user and attempt to access S3 buckets.
{% endstep %}
{% endstepper %}

Discussion Point: Why can/can’t this user perform actions on other services?

***

Exercise 2: Using IAM Groups

{% stepper %}
{% step %}
### Create groups

* Create two groups: “Admins” (full access) and “Developers” (limited S3 and EC2 read).
{% endstep %}

{% step %}
### Assign users

* Move users into each group.
{% endstep %}

{% step %}
### Demonstrate effects

* Demonstrate permission changes by moving users between groups.
{% endstep %}
{% endstepper %}

***

Exercise 3: Creating and Assigning Custom Policies

{% stepper %}
{% step %}
### Write a policy

* Write a custom policy (e.g., only allowing access to a specific S3 bucket).
{% endstep %}

{% step %}
### Attach the policy

* Attach this policy to a user or group.
{% endstep %}

{% step %}
### Test

* Test the permissions in real time.
{% endstep %}
{% endstepper %}

***

Exercise 4: IAM Roles and Role Switching

Scenario: An application on EC2 must access DynamoDB, but no hardcoded credentials are allowed.

{% stepper %}
{% step %}
### Create the role

* Create a role with DynamoDB permissions.
{% endstep %}

{% step %}
### Attach role to EC2

* Attach the role to an EC2 instance.
{% endstep %}

{% step %}
### Demonstrate

* Show how the EC2 instance now has permissions, with no credentials stored.
{% endstep %}
{% endstepper %}

***

Exercise 5: Enforcing MFA (Multi-Factor Authentication)

{% stepper %}
{% step %}
### Enable MFA

* Enable MFA for a sensitive user (e.g., an admin).
{% endstep %}

{% step %}
### Login process

* Show the process of login with MFA.
{% endstep %}
{% endstepper %}

***

### 4. Real-Time Scenarios To Discuss During Practice

* Restricting developers from deleting production resources.
* Granting temporary access to a third-party auditor.
* Allowing an application running on AWS Lambda to access only one S3 bucket.
* Implementing least privilege for sensitive financial data.

***

### Scenario-Based Questions

Use these to encourage critical thinking. Expand each question to discuss answers during the session.

<details>

<summary>1. A developer needs access to update code in a CodeCommit repository, but must not have delete permissions. How would you implement this with IAM?</summary>



</details>

<details>

<summary>2. You want to allow an external consultant to access only specific logs in CloudWatch for 7 days. What’s the best approach?</summary>



</details>

<details>

<summary>3. A web app running on an EC2 instance must upload files to an S3 bucket: How do you grant permissions without using static credentials?</summary>



</details>

<details>

<summary>4. If an employee leaves, explain steps to immediately revoke all their AWS access.</summary>



</details>

<details>

<summary>5. Describe how you would force MFA for all users accessing the AWS Management Console.</summary>



</details>

<details>

<summary>6. What is the best way to control permissions for different levels of access (e.g., read-only, admin) across many users?</summary>



</details>

<details>

<summary>7. What problems might arise if you assign ‘AdministratorAccess’ to all users, and how would you avoid it?</summary>



</details>

***

AWS Identity and Access Management (IAM) is like the security guard of the cloud—it decides who can do what in which part of your AWS account.

### Four Building Blocks

| Component  | Think of it as …           | Key Purpose                                                                                     | Where Policies Attach               |
| ---------- | -------------------------- | ----------------------------------------------------------------------------------------------- | ----------------------------------- |
| **User**   | A named employee badge     | Gives one person or an app a login and keys                                                     | Directly to the user                |
| **Group**  | A department badge drawer  | Lets you give the same permissions to many users at once                                        | To the group (flows to all members) |
| **Role**   | A temporary visitor badge  | Grants short-lived permissions that anyone (user, service, or account) can “put on” when needed | To the role                         |
| **Policy** | The rule sheet on the wall | JSON document that lists allowed/denied actions, resources, and conditions                      | Attaches to users, groups, or roles |

IAM Components at a Glance

{% hint style="info" %}
Why it matters:

* **Least privilege:** start with zero access; add only what’s required.
* **Auditability:** each badge swipe (API call) is logged, so you know who did what.
* **Flexibility:** switch roles instead of sharing long-term passwords.
{% endhint %}

### Story-Based Explanation

{% stepper %}
{% step %}
### New intern joins (User)

Create an IAM user “intern-arya” — she gets her own username, console password, and OPTIONAL access keys for CLI.
{% endstep %}

{% step %}
### Intern joins “Developers” (Group)

Instead of attaching 10 policies to Arya, add her to the “Developers” group that already has `AmazonEC2ReadOnlyAccess`. One change updates everyone in the group.
{% endstep %}

{% step %}
### EC2 instance uploads logs (Role)

An EC2 server cannot store passwords safely. Instead, it assumes the role `EC2-S3-LogWriter` that holds `PutObject` rights to a specific S3 bucket. AWS automatically rotates the temporary keys every few hours.
{% endstep %}

{% step %}
### External auditor arrives (Cross-Account Role)

You create role `Audit-Review` that trusts the auditor’s AWS account for one week. No new user in your account, and the role auto-expires.
{% endstep %}

{% step %}
### New policy needed (Policy)

Security adds a JSON policy denying `s3:DeleteObject` in the production bucket. Attach it to the “Developers” group — instantly enforced for Arya and 49 other developers.
{% endstep %}
{% endstepper %}

***

### Mental Model Cheat-Codes

* **User = Person/App with long-term credentials**
* **Group = Container that hands down permissions**
* **Role = Borrowed identity with temporary keys**
* **Policy = If/Then rules for allow/deny**

Remember: _Groups cannot log in_ and _roles have no password_.

### Quick Analogy

| Real-World Office         | AWS IAM Parallel |
| ------------------------- | ---------------- |
| Employee ID card          | IAM User         |
| Department list (IT, HR)  | IAM Group        |
| Visitor pass / Master key | IAM Role         |
| Building access matrix    | IAM Policy       |
