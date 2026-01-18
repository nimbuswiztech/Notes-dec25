# Difference Between Roles and Policies in AWS

> **The difference between IAM roles and policies in AWS is that a role is a type of IAM identity that can be authenticated and authorized to utilize an AWS resource, whereas a policy defines the permissions of the IAM identity.**

Keeping your cloud computing infrastructure secure is critical to preventing unauthorized users from gaining access to company data and protecting your AWS resources. IAM technology makes it easy to ensure that the right people have the right access to the right resources at the right time by allowing users to confirm their identities using a single set of login credentials.&#x20;

With the right IAM solution, users are able to move seamlessly between applications to streamline their workflows—preventing disruptions from cumbersome authentication protocols while keeping security tight.[<br>](https://www.strongdm.com/ebooks/identity-and-access-management-ebook)

### What is an AWS IAM Policy? <a href="#what-is-an-aws-iam-policy" id="what-is-an-aws-iam-policy"></a>

IAM is a framework of policies and technologies to ensure that the right users have the appropriate access to technology resources. An [AWS IAM policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html) defines the permissions of an identity (users, groups, and roles) or resource within the AWS account. An AWS IAM policy regulates access to AWS resources to help ensure that only authorized users have access to specific digital assets. Permissions defined within a policy either allow or deny access for the user to perform an action on a specific resource.

IAM policies can either be identity-based or resource-based. Identity-based policies are attached to an identity (a user, group, or role) and dictate the permissions of that specific identity. In contrast, a resource-based policy defines the permissions around the specific resource—by specifying which identities have access to a specific resource and when. Identity-based policies and resource-based policies both uphold IAM security standards and give organizations flexibility with how to best set up their resource management strategy to meet their needs.

#### What makes up an actionable AWS IAM policy statement?

A series of elements attributed to a broader policy can take us from general policy permissions to specific resource actions. Taken together, these elements make up a policy statement (the main policy element) that provides granular information regarding a single permission.&#x20;

* **Services:** The first “level” of the broader policy statement, AWS IAM either recognizes or does not recognize a specific service, like Amazon EC2 or Amazon S3. If it does not recognize the service, it will be classified as “uncategorized.” If it is recognized, IAM will either “Allow” or “Deny” the service based on the effect of the policy.&#x20;
* **Actions:** Actions can be recognized or unrecognized by the IAM. If recognized, the action is included under “List”, “Read”, “Write”, or “Permissions Management” access levels. These access-level classifications define the level of access the action grants a user according to policy permissions.&#x20;
* **Resources:** Provides a specific key, condition, or value that defines whether the action supports a resource-level permission. It is important to note that actions and resources must be compatible. If a specified resource is not valid for an action as defined by the IAM, then any request to use the action for the resource will fail.&#x20;
* **Conditions: Condition elements use condition operators (greater than, less than, equal to, etc.) to provide further constraints on actions within resources when a policy is in effect.**&#x200D;

While condition elements are optional within the overall policy statement, they can help generate a permission structure that [limits access to the bare minimum needed](https://www.strongdm.com/blog/aws-console-alternative-secure-aws-management) for a user to complete a specific task. This helps to drive optimal security for the AWS resources in use across the organization. &#x20;

### Example of an AWS IAM Policy <a href="#example-of-an-aws-iam-policy" id="example-of-an-aws-iam-policy"></a>

An organization initiates a short-term project. Current and new employees will need specific access to specialized resources during the lifecycle of this project. By creating an AWS IAM policy, IT admins can ensure that members of a project will only have access to the exact resources they’ll need to complete the project.&#x20;

They can do this by creating a policy that enables access to a particular resource for a specific date range and applying the policy to each IAM identity (users, roles, or groups).

For example:

&#x20;The policy stipulates that access to a particular resource is allowed on all dates except between April 1, 2022 and June 30, 2022. In order to use this policy, the "action" statement needs to be specified by an admin, then you assign that action to an identity (user, group, or role).

### What is an AWS IAM Role? <a href="#what-is-an-aws-iam-role" id="what-is-an-aws-iam-role"></a>

IAM identities provide access to resources under specific conditions within an AWS account. The purpose of an identity is not only to provide access to resources, but to ensure that users are authenticated and authorized so that your digital resources can be properly managed and remain secure. Identities come in three varieties: users, groups, and roles.&#x20;

[Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) are designed so that a set of permissions can easily be delegated to users on an individual basis. For example, instead of assigning an individual all their necessary permissions one at a time, they can be assigned a specific role that contains all the necessary permissions in a single step.&#x20;

Once a role is created, it can be assigned to as many individuals as needed. This makes roles particularly useful when assigning permissions to new users or changing permissions to users who have shifted jobs within their organization.

When it comes to AWS roles vs. policies, a good rule of thumb is to remember that policies are applied to roles, which can then be assigned to individual users via roles.

### Example of an AWS IAM Role  <a href="#example-of-an-aws-iam-role" id="example-of-an-aws-iam-role"></a>

An organization undergoes major expansion to undertake a new project: new employees are coming in, and current employees are shifting positions laterally within the organization. The current employees no longer need access to some of their old permissions but need access to new permissions. Additionally, the new employees need access to a wide variety of permissions to do their jobs. In order to accommodate this rapid new growth, IT administrators need a way to quickly and easily control access to their cloud resources while keeping their infrastructure secure.

Enter AWS IAM roles. Administrators create roles that are tied to specific policies that are appropriate for the new project. Employees—both current and new employees—can then be easily assigned to their new roles so that they only have access to what their new positions require.&#x20;

### The Difference Between AWS IAM Users, Groups, and Roles <a href="#the-difference-between-aws-iam-users-groups-and-roles" id="the-difference-between-aws-iam-users-groups-and-roles"></a>

As previously mentioned, users, groups, and roles are all specific types of [AWS identities](https://docs.aws.amazon.com/IAM/latest/UserGuide/id.html). Identities can be authenticated and authorized to perform actions using AWS resources under certain conditions.&#x20;

#### IAM User vs. IAM Role

An IAM user is a single person or service entity/application that interacts with AWS resources through service requests and modifications. AWS users consist of a name, password, and a pair of unique API access keys that grant them permissions according to policy condition criteria established by an administrator.&#x20;

The difference between an IAM role and a user is that a role can be temporarily or permanently applied to a user to give the user bulk permissions for a task. Unlike a user, a role does not have associated passwords or credentials and can be easily applied to multiple users to grant access to a set of permissions at once.&#x20;

#### AWS Groups vs. Roles

An AWS group is simply a collection of multiple users. When changes are made to the permissions of the group, the changes affect each individual user within that group. Policies can be attached directly to groups, so there is no need to assign permissions on an individual basis if they are applicable to the entire group. Moving users between groups can attach appropriate permissions when necessary, instead of editing permissions for a single user.&#x20;

<br>
