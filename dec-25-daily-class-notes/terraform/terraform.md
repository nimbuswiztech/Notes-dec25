# terraform

Terraform is an infrastructure as code (IaC) tool that lets you build, change, and version cloud and on-prem resources safely and efficiently. What does Infrastructure as Code mean? It means instead of manually creating resources, we declare the resources needed as code. This code will handle resource creation, updates, and deletion.

## Benefits

* Open source, platform agnostic (multiple cloud platforms)
* Terraform is Declarative
* Terraform is Agentless
* Collaboration
* The human-readable configuration language

## How does Terraform work?

Terraform creates and manages resources on cloud platforms and other services through their application programming interfaces (APIs). Providers enable Terraform to work with virtually any platform or service with an accessible API. HashiCorp and the Terraform community have already written thousands of providers to manage many different types of resources and services.

## Provider

Providers are the plugins that Terraform uses to manage those resources. Every supported service or infrastructure platform has a provider that defines which resources are available and performs API calls to manage those resources. So, providers understand API interactions and expose resources.

* Registry — directory of providers: https://registry.terraform.io
* Provider — understands API interactions and exposes resources
* `terraform init` — download provider

## The core Terraform workflow

(Diagram or image omitted)

## Authentication — AWS

(Diagram or image omitted)

## Write configuration

* HashiCorp Configuration Language (HCL)
* Declarative (desired state = actual state)
* Easy to read and write
* A JSON-like syntax with `.tf` extension
* Example: `main.tf`

Configuration file contains resource blocks, provider configuration, variables, outputs, etc.

## Resource block {}

(Examples / diagrams omitted)

## Authentication — Azure

(Diagram or image omitted)

## Variables

A variable in Terraform is similar to a variable in a programming language. A variable is a container that holds a value. It can be used to represent different values at different times during the execution of a program. Variables in Terraform are a great way to define centrally controlled reusable values. Terraform supports three different types of variables: input variables, output variables, and local variables.

* Input variables = like function arguments
* Output variables = like function return values
* Local variables = like a function’s temporary local variables

A variable is declared using the `variable` keyword and referenced using `var.variableName`.

Types of values that can be accepted by variables: string, number, bool, list, set, map, object, tuple.

### Assigning values to variables

* Command line argument
  * Example: `terraform plan –var rgname=“demo123“ –var rglocation=“East US”`
* Using file (`.tfvars`)
  * Example: `terraform apply -var-file prod.tfvars`
* Files auto-loaded by Terraform
  * `terraform.tfvars` or `terraform.tfvars.json`
  * `.auto.tfvars` or `.auto.tfvars.json`
* Environment variables
  * `TF_VAR_variableName`

## Local Variable

A local value assigns a name to an expression, so you can reference the local multiple times, reducing duplication in your code. Unlike input variables, locals are not set directly by users of your configuration. Local variables are declared using the `locals` block. Once a local value is declared, you can reference it as `local.name`.

## Output Variable

Output variables can be used to print certain values in the CLI output after running `terraform apply`. A child module can use output variables to expose a subset of its resource attributes to a root module.

## Data Source

Data sources let you dynamically fetch data that is present outside the Terraform context. Examples: AWS account number, AMI ID, list of availability zones, secrets. Use data sources to make your configuration more dynamic. You can fetch AWS availability zone data from a data source instead of hardcoding it in your config.

It’s a two-step process:

1. Declare the data source
2. Access the data source

Each infrastructure configuration in Terraform has resource blocks and data source blocks. Like resources, data sources have arguments and attributes for reference.

(Example diagrams omitted)

## Meta-arguments of resource {}

Meta-arguments in Terraform are special arguments used with resource blocks to control behavior or influence provisioning. They provide additional configuration beyond resource-specific arguments.

* `depends_on`
* `count`
* `for_each`
* `provider`
* `lifecycle`

(Reference: https://developer.hashicorp.com/terraform/language/meta-arguments/depends\_on)

## State

Terraform state is a vital mechanism that records metadata about your infrastructure. This includes not just the configurations you’ve written but also identifiers, dependencies, and other relevant details of the resources that Terraform manages. Essentially, the state is a snapshot of your infrastructure at a given point in time, helping Terraform predict and execute changes efficiently. Mapping of your desired state (config files) to real world resources is done through the state file. It is the single source of truth for Terraform.

Terraform uses state to determine which changes to make to your infrastructure. Prior to any operation, Terraform does a refresh to update the state with the real infrastructure. (See: https://developer.hashicorp.com/terraform/cli/commands/refresh)

By default, the state file is stored locally as `terraform.tfstate` (a JSON file). Terraform recommends storing state remotely to version, encrypt, and securely share it with a team. A `backend` block in the configuration determines where Terraform stores its state file. Backends are responsible for supporting state locking if possible. Not all backends support locking.

State locking prevents concurrent runs of Terraform against the same state. With state locking, Terraform will lock your state for all operations that could write state. This prevents others from acquiring the lock and potentially corrupting your state. Examples of backends that support state locking: AWS S3 (with DynamoDB for locking), Postgres, Google Cloud Storage, AzureRM. Follow backend documentation for details. (See: https://developer.hashicorp.com/terraform/language/state/locking)

Terraform state can contain sensitive data (resource IDs, database passwords, private keys, etc.). When using local state, it is stored in plain-text JSON files. Storing state remotely can provide better security. Terraform recommends storing state remotely with encryption at rest. For example, the S3 backend supports encryption at rest when the `encrypt` option is enabled. IAM policies and logging can be used to identify any invalid access. Another option is using Terraform Cloud, which always encrypts state at rest.

## Workspaces

Terraform workspaces are associated with a specific working directory and isolate multiple state files in the same working directory, letting you manage multiple groups of resources with a single configuration file. Terraform starts with a single workspace named `default` that you cannot delete. When you run `terraform plan` in a new workspace, Terraform does not access existing resources in other workspaces. These resources still physically exist, but you must switch workspaces to manage them. A common use for multiple workspaces is to create a parallel, distinct copy of infrastructure to test a set of changes before modifying production infrastructure.

### Workspaces CLI commands

```bash
$ terraform workspace [list, show, new, delete, select]
```

## Terraform Module

Modules in Terraform are the same concept you have learned in programming languages. Modules are the key ingredient to write reusable, maintainable, and testable Terraform code. Modules are containers for multiple resources that are used together. A module consists of a collection of `.tf` files kept together in a directory.

* A Terraform module (usually the root module) can call other modules to include their resources into the configuration.
* A module that has been called by another module is called a child module.
* A child module can be included within the root module using a `module` block, which specifies the `source` of the module and any input variables that need to be set.
* The root module is the top-level module of the Terraform config. It serves as the entry point of your entire Terraform configuration and typically includes provider configuration and any variables needed to define your infrastructure resources. The root module is responsible for initializing Terraform, configuring providers, and declaring resources.

When you browse the Terraform Registry you will find publicly available modules (almost 16 thousand modules). You can use these modules to create resources or create your own modules from scratch.

(Direct documentation excerpts and diagrams omitted)

## Terraform import

Terraform `import` is used to bring manually created existing infrastructure under Terraform management. Importing existing resources into Terraform ensures that your infrastructure’s state is centralized and managed through a single source of truth, enhancing visibility and control. Once imported, your resources integrate with the broader Terraform ecosystem, allowing you to leverage modules, variables, and other features. Terraform import is available in Terraform v1.5.0 and later.

{% stepper %}
{% step %}
### Old way

The old way to import resources under Terraform management is using the CLI command:

terraform import \<resource\_type>.\<resource\_name> \<resource\_id>

This only updates the state file. You need to manually update the configuration file to match the imported resource.
{% endstep %}

{% step %}
### New way (import block with manual resource block)

The new way is to use an `import` block. This block accepts two arguments: `to` and `id`. You still need to add the corresponding `resource` block manually. After you do `terraform init`, `terraform plan`, and `terraform apply`, you comment out the `import` block.
{% endstep %}

{% step %}
### Newer way (auto-generate config)

This approach will auto-generate the configuration file. You declare an `import` block without a `resource` block, then run `terraform plan` with the generate config flag. This will auto-generate the configuration file. Review the generated file and run `terraform plan` and `terraform apply`. After `apply`, comment out the `import` block.
{% endstep %}
{% endstepper %}

## Closing

Please follow me on YouTube for the detailed explanation as well as hands-on demos!

HAPPY LEARNING HAPPY CODING
