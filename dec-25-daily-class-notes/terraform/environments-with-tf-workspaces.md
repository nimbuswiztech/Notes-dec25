# environments with tf workspaces

Terraform workspaces provide an efficient way to manage multiple environments (e.g., development, staging, production) using the same Terraform configuration. This article guides you through the basics of Terraform workspaces and demonstrates their usage with practical commands.

### What are Terraform Workspaces?

Terraform workspaces are an organizational feature that allows you to maintain separate state files for different environments within a single configuration. This is useful for managing different stages of your infrastructure lifecycle, such as development, staging, and production, without needing to maintain separate directories or configurations for each environment.

### Setting Up and Managing Workspaces

{% stepper %}
{% step %}
### List Existing Workspaces

To see the list of existing workspaces in your current Terraform configuration, use the following command:

```
terraform workspace list
```

This command displays all the workspaces, with an asterisk (\*) indicating the currently active workspace.
{% endstep %}

{% step %}
### Create a New Workspace

To create a new workspace, use the following command:

```
terraform workspace new dev
```

In this example, a new workspace named `dev` is created. This workspace will have its own state file, separate from other workspaces.
{% endstep %}

{% step %}
### Verify the New Workspace

After creating a new workspace, you can list the workspaces again to verify its creation:

```
terraform workspace list
```
{% endstep %}

{% step %}
### Using `terraform.workspace`

In Terraform, `terraform.workspace` is used to get the current workspace's name. This can be particularly useful for naming resources or for outputs to dynamically reflect the active workspace. For example:

```
output "current_workspace" {
  value = terraform.workspace
}
```

`terraform.workspace` is used within the `tags` block to dynamically name the instance based on the current workspace.
{% endstep %}
{% endstepper %}

### Example Workflow

Here’s an example workflow that demonstrates creating, selecting, and managing workspaces:

```hcl
provider "aws" {
  region = "ap-south-1"
}

resource "aws_instance" "myinstance" {
  ami           = "ami-00fa32593b478ad6e"
  instance_type = "t2.micro"
  tags = {
    Name = "${terraform.workspace}-instance"
  }
}

output "outputtest" {
  value = aws_instance.myinstance.public_ip
}
```

Instance Name varies based on the Terraform workspace, using `{terraform.workspace}` for dynamic naming.

{% stepper %}
{% step %}
### Create new workspaces `dev` and `prod`

```
terraform workspace new dev
terraform workspace new prod
```
{% endstep %}

{% step %}
### Run the terraform script for `prod` workspace

```
terraform apply -auto-approve
```

The instance created will be tagged as `prod-instance`.
{% endstep %}

{% step %}
### Switch to `dev` workspace

```
terraform workspace select dev
```
{% endstep %}

{% step %}
### Run the terraform script for `dev` workspace

```
terraform apply -auto-approve
```

The instance created will be tagged as `dev-instance`.
{% endstep %}
{% endstepper %}

Using `terraform.workspace` makes your Terraform configurations more flexible and environment-aware, allowing for smoother and more organized infrastructure management across multiple environments.

### Conclusion

Terraform workspaces provide a powerful way to manage multiple environments within a single Terraform configuration. By using workspaces, you can isolate state files and ensure that changes in one environment do not affect another.

This article covered essential commands for listing, creating, switching, applying configurations, and destroying resources within Terraform workspaces. Leveraging these capabilities can greatly enhance your infrastructure management process, providing a clean and efficient way to handle different environments.

Happy managing!
