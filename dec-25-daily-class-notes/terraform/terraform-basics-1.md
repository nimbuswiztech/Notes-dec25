# terraform basics 1

**Introduction:**

Terraform is an open-source infrastructure provisioning tool created by HashiCorp. It allows you to define and manage your infrastructure using a declarative language which is written in GO language.

Terraform allows you to create, modify, and destroy infrastructure resources across different cloud providers, such as AWS, Azure, GCP, etc. It describes the desired state of the infrastructure on the premises environment.

### Benefits of Using Terraform

🔥 It supports multiple cloud providers such as AWS, Azure, Oracle, GCP, and more.

🔥 Terraform configuration is written in an easy-to-understand language.

🔥 It simplifies the management and orchestration of multi-tier infrastructure.

🔥 Terraform modules allow you to separate resources in dedicated and reusable templates.

***

### Terraform Lab setup

To make a terraform lab setup we have to install the following pre-requisites:

1. Download & install [terraform.](https://developer.hashicorp.com/terraform/downloads)
2. IDE (used: [VS code editor](https://code.visualstudio.com/download)).
3. Terraform Extensions for VS Code Editor:
   * Terraform Autocomplete
   * AWS Tool Kit
   * Simple Terraform Snippets
   * AWS CLI
4. Configure AWS Credentials:
   * AWS Account
   * Generate Security Credentials
   * Configure in AWS CLI

***

### Terraform Configuration

* Terraform script/code are stored in plain text files with a .tf extension.
* The file that contains terraform code is called a configuration file or terraform manifest.
* You should know your working directory before executing the code; whenever you execute Terraform code you should be in the working directory where the configuration file is available.

### Terraform Configuration Syntax

```
<BLOCK TYPE> "<BLOCK LABEL>" "<BLOCK LABEL>"   {
  # Block body
  <IDENTIFIER> = <EXPRESSION> # Argument
}
```

### Blocks and Arguments

* Blocks are like a container to define other contents of your infrastructure.
* An argument assigns a value to a particular name: argument name and argument value.
* Argument names, block type names, and the names of most Terraform-specific constructs like resources, input variables, etc. are all identifiers.

> Let’s start with terraform basic concepts in terraform registry, providers and resources and terraform workflow.

**Registry** has all the reusable components for providers and modules. For more information check the website [registry](https://registry.terraform.io/). It provides official, verified, and community tier providers for free in terraform configuration.

**Providers** are plugins that implement resource types. It’s a logical abstraction of an upstream API. They are responsible for understanding API interactions and exposing resources.

**Modules** make your work easy, as we can just plug in the common modules to our terraform code. Modules are self-contained packages of Terraform configurations that are managed as a group.

***

### Terraform Workflow

The workflow consists of lifecycle stages which start with init, plan, apply, and destroy. All these are executed as commands for the written code to update or change the infrastructure built.

{% stepper %}
{% step %}
### Terraform Init

🎯 Terraform initializes the working directory containing terraform configuration files.

🎯 Terraform downloads and installs the provider’s plugin so that it can later be executed.

🎯 Terraform creates a lock file .terraform.lock.hcl to record the provider selections it made. It also initializes the backend configuration.
{% endstep %}

{% step %}
### Terraform fmt

🎯 The terraform fmt command is used to rewrite Terraform configuration files to a canonical format and style.
{% endstep %}

{% step %}
### Terraform Validate

🎯 The terraform validate command validates the configuration files in a directory.

🎯 It verifies whether a configuration is syntactically valid and is primarily useful for general verification of reusable modules, including the correctness of attribute names and value types.

🎯 It can run before terraform plan. Validation requires an initialized working directory with any referenced plugins and modules installed.
{% endstep %}

{% step %}
### Terraform Plan

🎯 The terraform plan command is used to create an execution plan. It will not modify infrastructure. It compares the desired Terraform state with the current state in the cloud.

🎯 This command is a convenient way to check whether the execution plan for a set of changes matches your expectations without making any changes to the state.
{% endstep %}

{% step %}
### Terraform Apply

🎯 It applies the changes required to reach the desired state of the configuration. This command executes the plan that is already created.

🎯 It will write data into terraform.tfstate file. Once the apply is completed, resources are immediately available.
{% endstep %}

{% step %}
### Terraform Destroy

🎯 The terraform destroy command is used to delete the created resources in the Terraform-managed infrastructure.
{% endstep %}
{% endstepper %}

***

_In the terraform configuration file, most terraform features are controlled by top-level blocks. We need to have the following 3 blocks for creating a terraform configuration._

### Terraform Block

Here is the required terraform version, list of providers required, and terraform backend.

```
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "5.0.1"
    }
  }
}
```

### Provider Block

It declares providers for Terraform to install the providers which are going to be used. Terraform relies on the providers to interact with remote systems. Provider configuration belongs to the root module.

```
provider "aws" {
  region  = "us-east-1"
  profile = "default"
}
```

### Resource Block

Each resource block describes one or more infrastructure objects. Each resource block is an infrastructure object, like compute instances, virtual networks, or other components.

```
resource "aws_instance" "webserver" {
  ami           = "ami_id"
  instance_type = "t2.micro"
}
```

Let us apply the above terraform blocks in configuration and execute commands.

***

### Creating an EC2 Instance on AWS using Terraform

(Images removed)

***

### Important files created during execution

#### 1. Dependency Lock File

* The dependency lock file belongs to the configuration as a whole, rather than to each separate module in the configuration.
* The lock file is named **.terraform.lock.hcl** and is available in the subdirectory of .terraform of your working directory.
* It automatically gets created when we execute the `terraform init` command to record the provider so that Terraform can make the same selections by default when you run `terraform init` in the future.

#### 2. Terraform State File

* Terraform state stores the infrastructure information and its configuration. The state file maintains a record of the resources created by your configuration and maps them to real-world resources.
* The state file is stored in JSON format in the name of **terraform.tfstate** by default in our local workspace. Also, Terraform maintains a backup file for terraform state which is named **terraform.tfstate.backup**.
* Whenever we execute `terraform apply` a new terraform.tfstate file will be created and its backup will be written in terraform.tfstate.backup file.
* The state file can be stored/maintained remotely in various methods like using a version control tool Git, Amazon S3 as remote store and also in Terraform Cloud, Enterprise, and Consul.

***

### Terraform State commands

These are the list of terraform state commands used frequently.

* `terraform state list` — Lists the resources within the state file.
* `terraform state mv` — Move items within terraform state. Used for resource renaming without destruction.
* `terraform state pull` — Manually downloads and outputs the state from the state file.
* `terraform state push` — Manually upload a local state file to the remote state.
* `terraform state rm` — Removes items from the state & no longer managed by Terraform. The resources removed from the state are not physically destroyed.
* `terraform state show` — Show attributes of a single resource in the state.

***

### What Is the Current State and Desired State in Terraform

#### Current state

* Current state is the present state of the resources deployed, stored locally as terraform.tfstate file and not in remote infrastructure.
* After creating the infrastructure, any manual changes made in the infrastructure will not reflect in the current state file.
* In the next execution of the `terraform apply` command, Terraform will reconcile to keep the infrastructure the same as represented by the current state unless the configuration changed.

#### Desired state

* Desired state is the new state expected to be applied from your terraform configuration.
* It is compared with the current state every time to update the infrastructure.
* It will detect the changes between the desired state and the current state and try to ensure that the deployed infra matches the desired state.
* If there is a difference between the two, `terraform plan` will show the necessary changes required to achieve the desired state.

***

### Types of Terraform state store

There are two types of stores for Terraform state: local and remote.

* Local state refers to the Terraform state stored on the local filesystem, i.e. on your laptop or whatever system you are running the Terraform command from.
* Remote state is Terraform state stored remotely, such as in an S3 bucket or a database like PostgreSQL.

***

### Terraform Variables

Terraform variables store values which can be reused in terraform configuration. The functions of variables are as follows:

* Input Variables — function arguments, so users can customize behavior without editing the source.
* Output Values — like return values for a Terraform module.
* Local Values — a convenience feature for assigning a short name to an expression.

#### Input Variables

Input variables are declared in the `variable` block. Common arguments: name, type, default, and description.

Example structure:

```
variable "<variable name>" {
  description = "updating instance type"   # Give a meaningful description
  type        = string                      # ex: string, number, bool, list, map
  default     = "t2.micro"                  # variable default value
}
```

Basic types:

* String — A sequence of Unicode characters representing some text
* Number — A numeric value that includes either a whole number or a fractional value.
* Bool — TRUE or FALSE

Complex types:

* Map — associative arrays used to look up one value from another (e.g., region -> AMI).
* List — ordered collection accessed by index.

***

### Assigning Values to Input Variables

There are multiple ways to assign values for input variables:

* In command-line option
* In variable definitions files
* As an environment variable

Command-line Option:

To specify individual variables on the command line, use the `-var` option while running terraform plan/apply. Example:

```
$ terraform plan -var 'aws_region=us-west-1'
```

Authentication example:

```
$ terraform apply –var="AWS_ACCESS_KEY=<Access key>"
$ terraform apply –var="AWS_SECRET_KEY=<Secret key>"
```

#### CLI Variables

You can also pass variables via `-var`:

```
$ terraform apply -var="image_id=ami-xyz234"
```

#### Variable Definition Files (tfvars)

To set many variables in a single file, use a variable definition file ending in `.tfvars` or `.tfvars.json`. Example:

```
$ terraform apply –var-file="sample.tfvars"
```

Auto tfvars files:

* Files ending in `.auto.tfvars` or `.auto.tfvars.json` are loaded automatically without specifying `-var-file`.

#### Environment Variables

Terraform searches environment variables named `TF_VAR_` followed by the name of the declared variable. Environment variable names are case-sensitive and must match the variable name exactly.

Example:

```
Export TF_VAR_AWS_REGION=us-east-1
```

***

### Output Values

Output values make information about your infrastructure available on the command line and can expose information for other Terraform configurations to use. Output values are similar to return values in a programming language.

***

### Local Values

Terraform local values are like temporary local variables. They assign a name to an expression so it can be used multiple times within the same module, helping to avoid repeating the same values.

***

### Meta Arguments in Terraform

Meta arguments are used to control how resources are created, updated, and destroyed. They control the behavior of resources.

Common meta-arguments:

* count
* count.index
* for\_each
* depends\_on
* provider
* lifecycle

Examples:

Count:

```
resource "aws_instance" "sample" {
  count         = 4
  ami           = "ami-ID"
  instance_type = "t2.micro"
}
```

Count.index:

```
resource "aws_instance" "linux_server" {
  count         = 4   # creates four similar EC2 instances
  ami           = "ami-ID"
  instance_type = "t2.micro"
  tags = {
    Name = "linux_server ${count.index}"
  }
}
```

For\_each:

```
resource "aws_instance" "webserver" {
  for_each      = var.webservers
  ami           = "ami-ID"
  instance_type = "t2.micro"
  tags = {
    name = each.value["name"]
  }
}
```

Depends\_on:

* Used to specify that the resource depends on another resource (useful when one resource must exist before another).

Lifecycle:

* Manages the lifecycle of resources in the configuration (sub-arguments include create\_before\_destroy, prevent\_destroy, ignore\_changes).

Example:

```
resource "aws_instance" "example" {
  lifecycle {
    create_before_destroy = true
  }
}
```

***

### Terraform Provisioners

Provisioners are used to run specific actions on the local machine or on a remote machine to prepare servers or other infrastructure objects for service. There are two types: generic and vendor-specific (chef, puppet).

Common provisioners:

* file
* local-exec
* remote-exec

File provisioner example:

```
provisioner "file" {
  source      = "specify the path from where we need to copy"
  destination = "destination path where you want your file to be"
}
```

Local-exec provisioner:

* Executes commands locally after the resource is created.

Remote-exec provisioner:

* Executes scripts on the remote resource (e.g., via SSH).

***

### Terraform Modules

Terraform modules allow you to create a logical abstraction on some resource sets. They allow you to group resources and reuse them.

#### Root Module

* Each terraform configuration has at least one module known as the root module, which consists of the resources defined in the .tf files in the main working directory.

#### Child Modules

* A root module can call other modules (child modules) to include their resources into the configuration.
* Child modules can be called multiple times within the same configuration, and multiple configurations can use the same child module.

#### Published Modules

* Terraform Registry hosts a broad collection of publicly available Terraform modules for configuring many kinds of common infrastructure.
* Terraform can download them automatically if you specify the appropriate source and version in a module call block.
* Terraform can load modules from a public or private registry. These modules are free to use.

***

### How to Use Modules

* Module blocks document the syntax for calling a child module from a parent module, including meta-arguments like for\_each.
* Module sources document what kinds of paths, addresses, and URIs can be used in the `source` argument of a module block.
* The Meta-Arguments section documents special arguments that can be used with every module, including providers, depends\_on, count, and for\_each.

Example: reusable module for hosting a static website.

Module structure:

Terraform

* Modules
  * main.tf
  * variables.tf
  * outputs.tf
  * S3
    * main.tf
    * variables.tf
    * output.tf
    * www
      * index.html
      * error.html

S3 bucket resource example inside module:

```
resource "aws_s3_bucket" "staticsite" {
  bucket        = var.bucket_name
  acl           = "public-read"
  policy        = <<EOF
  # here paste the aws_iam_policy_document
  EOF
  website {
    index_document = "index.html"
    error_document = "error.html"
  }
  tags          = var.tags
  force_destroy = true
}
```

Call the module from root:

```
module "website_s3_bucket" {
  source      = "../modules/s3" # relative path
  bucket_name = var.my_bucket
  tags        = var.my_tags
}
```

Using the terraform module in this configuration hosts the static website.

(Execution screenshots removed)

***

### Terraform workspace

Workspaces allow you to create different and independent states on the same configuration. Terraform workspaces reduce code duplication. When compatible with a remote backend, workspaces can be shared with your team.

How to use Workspace:

```
$ terraform -help workspace
```

Usage (commands):

* new — Create a new workspace
* list — Lists Workspaces
* show — Shows the name of the current workspace
* select — Select a workspace
* delete — To delete a workspace

How it works:

* Multiple workspaces are supported by backends like S3, AzureRM, Remote, Consul, etc.
* Example: deploy the same configuration to development, staging, and production — one state per workspace. If deployed to all 3 environments, the workspace layout will produce three different state files:

Example tree:

├── website.sh\
├── instance.tf\
├── provider.tf\
├── terraform.tfstate.d\
│ └── dev\
│ └── terraform.tfstate\
│ └── Stag\
│ └── terraform.tfstate\
│ └── Prod\
│ └── terraform.tfstate\
├── terraform.tfvars\
└── vars.tf

***

### Workspace Setup — S3 as Remote Backend

You can back up the remote state of each workspace to S3 using a remote backend configuration.

(Images removed)

A workspace was created for the development environment and later cleaned up.

***

In conclusion, this guide has covered the fundamentals of Terraform configuration to help you get started with infrastructure-as-code.

Remember, continuous learning and staying up to date with the latest Terraform releases and best practices will enhance your proficiency and lead to even more successful infrastructure deployments.

Happy Terraforming :)
