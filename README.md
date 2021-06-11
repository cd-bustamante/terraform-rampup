# Terraform Ramp up

Terraform is a tool for building, changing, and versioning infrastructure safely and efficiently. Terraform can manage existing and popular service providers as well as custom in-house solutions.

It is as easy to start it only requires a binary to be executed and can be [downlowded from here](https://www.terraform.io/downloads.html).

It uses Hashicorp Configuration Language (HCL) to define and create your Infrastructure resources as Code - Infrastructure as Code (IaC) -

Terraform [Documentation](https://www.terraform.io/intro/index.html)

## Terraform Workflow

**Write** -> **Plan** -> **Apply**

### Write

Start with a repo as common best practices using version control to facilate collaboration. 

### Plan

Review changes that your code will make in your Infrastructure, these two steps of the workflow create a cycle to validate and correct until your code has the correct syntax for your resources you want to add/modify to your infrastructure.

### Apply

Step to provision your resources in your infrastructure

## Terraform commands

- **terraform init**

  - Downloads ancillary components (modules and plugins required to interact APIs)
  - Sets up backend for storing Terraform state file, which is a file used by TF to track resources configuration.

- **terraform plan**

  - Reads the code and creates and showas a plan of execution/deployment
  - Allows user to review plan before executing anything

- **terraform apply**

  - Deploys the instructions and statements in the code
  - Updates the deployment terraform state file.

- **terraform destroy**

  - Destroy all resources tracked by the state file.

## Terraform Code

- Terraform executes code in files with the .tf extention
- By default, Terraform looks for providers in the [Terraform providers registry](https://registry.terraform.io/browse/providers).

### [Terraform Documented Providers](https://registry.terraform.io/browse/providers)**

Providers are plugins to interact with your infrastructure provider APIs

```terraform
provider "Provider name" {
    parameter = "suppported parameters"
}
```

### Resource block (creates a resource)

```terraform
resource "TF provider resource name" "your given name to the resource" {
    parameter_1 = "your input"
    parameter_n = "your input"
}
```

referencing a resource within the code resource.my-resource

### Data source block (fetches config from an existing resource)

```terraform
data "TF provider resource name" "your given name to the resource" {
    parameter = "your input"
}
```

referencing a resource within the code data.my-data 

### Variables

Best practices to be defined in terraform.tfvars file

```terraform
variable "user_provided_based_types" {
    description = "custom config"
    type        = [string, number, bool]
    default     = "8080"
    validation (
      condition      = can(regex("8080|80", var.user_provided_based_types))
       error_message = "Port values can only be 8080 or 80"
    )
}
```

```terraform
variable "user-provided complex types" {
    description = "custom config"
    type        = [list, set, map, object, tuple]
    default     = ["list", "list2"]
    type        = list(object({
      internal = number
      external = number
      protocol = string
    }))
    default    = [
        {
          internal = 3535
          external = 1616
          protocol = "tcp"
        }
    ]
    validation (
      condition      = lenght(var.my-var) > 4
       error_message = "The string must be more than 4 characters"
    )
}
```

```terraform
variable "sensitive variable name" {
    description = "custom config"
    type        = string
    default     = "sensitive"
    sensitive   = true
    )
}
```

referencing a resource within the code var.my-var 

### Outputs

```terraform
output "workspace_id" {
    description = "Databrick workspace id"
    value = azurerm_databricks_workspace.ws.id
    )
}
```

```terraform
output "url" {
    description = "URL for the site"
    value = join(":"  ["http://localhost", tostring(var.external_port)])
    )
}
```

## Installing Terraform

1. Download binary, Unzip binary, place it on your system's path

2. Set up a Hashicorp Terraform repository on Linux, use package manager to install terraform, package manager installs and sets it up so that it is ready to use.

## Terraform Providers

- Plugins are release on a separate schedule than Terraform itself, hence each provider has its own series of version numbers.

- Best practice when working with Providers is to fix them to an specific version, so that any changes across provider version doesn't break your Terraform code.

## Terraform DEBUG

On your working machine from where you are using the Terraform Binary you need to set up environment variables such as:

```sh
export TF_LOG=TRACE
```

## Terraform State

Main Tool/Mechanism used by Terraform to map the resources defined in the code with the resources deploy in your managed infrastructure

Terraform.tfstate file is a json dump container containing all the metadata about your terraform deployment, and all details related your deployed resources in the managed infrastucture.

Can be stored locally or remotely.

Resource dependency metadata is also tracked via teh state file

Caches resource attributes for subsequent use to improve deployment performance.

**Never** lose your Terraform State file!

Save to local as default behaviour

### Terrafomr State Command

- Advance state management
- Listing out tracked resources and their details

#### Subcommands

- terraform state list - (list all resources)
- terraform state show - (show details of a resource)
- terraform state rm - (Delete a resource from TF state file, in other words the resources is not longer managed by TF)

### Remote State Storage

- Storage container such as AWS S3, Google Bucket, Azure Storage Container
- Allows collaboration, thanks to the functionality related to locking the state file to avoid parallel executions.
- Useful to share **output** values with other Terraform code.

## Terraform Provisioners

- TF way of bootstrapping custome scripts, commands or actions
- Can be run locally, or emotely on resources spun up though the TF deployment.
- In TF code, each individual resource can have its own provisioner defining the connection method (ex. SSH or WinRM) and the actions/commands or script to execute.
- Two main types of provisioners
  - Creation-time (used as default type)
  - Destroy0-time (add "when = destroy" inside of the provisioner code block)

### Best practices

- Hashicorp recommends using provisioners as a last resort, giving preference to try using inherent mechanism wihtin your infrastructure deployment to carry out custom tasks where possible.
- TF cannot track changes to provisioners.
- if the cmd within a provisioner returns non-zero return code, it's considered failed and underlying resource is tainted.

```terraform
resource "null_resource "dummy_resource" {
  provisioner "local-exec" {
    command = "echo '0' > status.txt"
  }
  provisioner "local-exec" {
    when    = destroy 
    command = "echo '1' > status.txt"
  }
  
}
```

```terraform
resource "aws_instance "ec2-vm" {
  ami                         = ami-1
  instance_type               = t2.micro
  key_name                    = aws_key_pair.master-key.key_name
  associate_public_ip_address = true
  vpc_security_group_ids      = [aws_security_group.test.id]
  subnet_id                   = aws_subnet.subnet.id
  provisioner "local-exec" {
    command = "aws ec2 wait instance-status-ok --region -us-west-1 --instance-ids ${self.id}"
  }
}

## Terraform Modules

Modules are containers for multiple resources that are used together. A module consists of a collection of .tf and/or .tf.json files kept together in a directory.

Modules are the main way to package and reuse resource configurations with Terraform.

Every Terraform configuration has at least one module, known as its **root module**, which consists of the resources defined in the .tf files in the main working directory.

A Terraform module (usually the root module of a configuration) can call other modules to include their resources into the configuration. A module that has been called by another module is often referred to as a child module.

Child modules can be called multiple times within the same configuration, and multiple configurations can use the same child module.

### Accessing modules

Can be downloaded from:

- Terraform Public Registry
- A private Registry

```terraform
module "consul" {
  source = "hashicorp/consul/aws"
}
```

Referencing a module

```terraform
resource "aws_elb" "example" {
  # ...

  instances = module.servers.instance_ids
}
```

## Terraform Built-In Functions

The Terraform language includes a number of built-in functions that you can call from within expressions to transform and combine values. The general syntax for function calls is a function name followed by comma-separated arguments in parentheses:

```terraform
<FUNCTION NAME>(<ARGUMENT 1>, <ARGUMENT 2>)
```

```terraform
resource "databricks_instance_pool" "nodes" {
  instance_pool_name = "DBSIP-${var.pooltype}-${var.lob}-${var.tier}"
  min_idle_instances = var.minidleinstances
  max_capacity       = var.maxcapacity
  node_type_id       = var.nodetypeid
  idle_instance_autotermination_minutes = var.idleautotermmin
  custom_tags = {
    LOB     = var.lob
    Tier    = join("-",["terraform", var.tier])
    UseCase = var.usecase
  }
```

```terraform
> contains(["cesar", "slalom", 3,], "3")
false
> contains(["cesar", "slalom", 3,], 3)
true
>
```

## Terraform Type Constraints

Terraform module authors and provider developers can use detailed type constraints to validate user-provided values for their input variables and resource arguments. This requires some additional knowledge about Terraform's type system, but allows you to build a more resilient user interface for your modules and resources.

Primitive Types (number, string, bool)
Complex Types (list, tuple, map, object)

- **Collection** types allow multiple values of one primitive types to be grouped together (ex):
  - list (type)
  - map (type)
  - set (type) 

```terraform
variable "collection" {
  type = list(number)
  default = [1,2,3]
}
```

- **Structural** types allow multiple values of different primitive types to be grouped together

```terraform
variable "structural" {
  type = object ({
    vmname = string
    vCPU   = number
  })
}
```

- **any** is a placeholder for a primitive type to have terraform match to the proper primitive type at the run time.

```terraform
variable "any" {
  type = list(any)
  default = [true, false, true]
}
```

## Dynamic Blocks

The goal is to have code cleaner by create code definition for repeatable nested configuration blocks inside of Terraform resources, can be used inside of data, provider, provisioner and resource blocks.

```terraform
variable "deploy_groups" {
  description = "Group to apply policy"
  type = list(string)
  default = [DataOps, DataAdmin]
}

resource "databricks_permissions" "can_use_admin_pool_nodes" {
  instance_pool_id  = databricks_instance_pool.nodes.id
  dynamic "access_control" {
    for_each = var.deploy_groups
      content {
        group_name       = access_control.key 
        permission_level = "CAN_MANAGE"
      }
  }
  ```

Best practices recommends to use dynamic blocks wehn you need to hide details such as when you are writing terraform modules.

## Terraform fmt, taint & import

### fmt

Formats Terraform code for readibility, best practices to use before pushing your code to version control  any time you made changes to the code.

```terraform
terraform fmt
```

### taint

Taints a resource, forcint it to be destroyed and recreated by modifying the state file, which causes the recreation workflow after next terraform apply execution

- Think about the dependencies when recreating a resourcer, ex External IP address on a VM

Useful scenarios, cause a provisioners to run, replace misbehaving resources forcefully or mimic side effects of recreation not modeled by any attributes of the resource. (third pary calls post creation)

```terraform
terraform taint resource_address
```

### import

Terraform is able to import existing infrastructure. This allows you take resources you've created by some other means and bring it under Terraform management.

Maps existing resources to TF using an "ID", which is dependent of the underlying vendor (ex. Databricks WS ID, VM ID, etc.)

Warning: Terraform expects that each remote object it is managing will be bound to only one resource address, which is normally guaranteed by Terraform itself having created all objects.

```terraform
terraform import resource_address ID
```

Useful cases could be when you are not in control of creation process of infrastructure and you are allow to manage them once are created.

### Terraform configuration blocks

Allows you to configure backend for storing state files, specifying TF version or TF provider version, enable TF experimental features, pass metadata to providers.

```terraform
terraform {
  required_verion = ">=0.13.0"
  required_providers {
    databricks = {
      source = "databrickslabs/databricks"
      version = ">=0.3.2"
    }
  }
}
```

## Terraform Workspaces (CLI)

- Terraform Workspaces are alternate state files within the same working directory
- By default Terraform provides a single workpace that is called default. This Workspace cannot be deleted.

```terraform
terraform workspace new/select <workspace-name>
```

Useful scenarios:, test changes using a parallel, distinct copy of infra (high cost), separate your different environments (PROD, DEV, ETC)

```terraform
resource "azurerm_linux_virtual_machine" "example" {
  count = terraform.workspace == "default" ? 5 : 1
  # ... other arguments
}
```

```terraform
resource "databricks_instance_pool" "nodes" {
  instance_pool_name = "DBSIP-${var.pooltype}-${var.lob}-${var.tier}"
  min_idle_instances = var.minidleinstances
  max_capacity       = var.maxcapacity
  node_type_id       = var.nodetypeid
  idle_instance_autotermination_minutes = var.idleautotermmin
  custom_tags = {
    LOB        = var.lob
    UseCase    = var.usecase
    Enviroment = ${terraform.workspace}
  }

  azure_attributes {}

  # Ignore attributes that are set by Azure for spot instances, these are causing update issues
  lifecycle {
    ignore_changes = [
      azure_attributes,
    ]
  }
}
```

## Terraform Debugging

Terraform uses environment variables to manipulate logging

### **TF_LOG** 

For enabling verbose logging. By default, it will send logs to stderr. The variable can be set to TRACE, DEBUG, INFO, WARN, ERROR. Being trace the most verbose level.

### **TF_LOG_PATH**

Persists logged output to an specifing file, giving a path.

```bash
export TF_LOG=TRACE
export TF_LOG_PATH=./terraform.log
```

## Terraform Cloud and Enterprise

### Hashicorp Sentinel - Policy as Code

- Enforces policies on your code using Sentinel language. It is trigger at the Terraform Plan command. 

Useful cases

- Organization policies for resource quotas
- Organization security policies

```sentinel
# This policy uses the Sentinel tfplan/v2 import to require that
# specified Azure resources have all mandatory tags

# Import common-functions/tfplan-functions/tfplan-functions.sentinel
# with alias "plan"
import "tfplan-functions" as plan

# Import azure-functions/azure-functions.sentinel
# with alias "azure"
import "azure-functions" as azure

# List of Azure resources that are required to have name/value tags.
param resource_types default [
  "azurerm_resource_group",
  "azurerm_virtual_machine",
  "azurerm_linux_virtual_machine",
  "azurerm_windows_virtual_machine",
  "azurerm_virtual_network",
]

# List of mandatory tags
param mandatory_tags default ["environment"]

# Get all Azure Resources with standard tags
allAzureResourcesWithStandardTags =
                        azure.find_resources_with_standard_tags(resource_types)

# Filter to Azure resources with violations using azurerm_virtual_machine
# Warnings will be printed for all violations since the last parameter is true
violatingAzureResources =
      plan.filter_attribute_not_contains_list(allAzureResourcesWithStandardTags,
                    "tags", mandatory_tags, true)


# Main rule
main = rule {
  length(violatingAzureResources["messages"]) is 0
}
```

## Terraform Vault

- Hashicorp Vault (secrets management software)
  - Dynamically provisions credentials and rotates them
  - Encrypts sensitive data in transit and at rest and provides fine-grained access to secrets using ACLs

1. Vault Admin stores long-lived credentials in Vault, configures permissions for temporary credentials when generated

2. Terraform operator integrate Vault using vault provider

3. At runtime TF reaches vault for temporary credentials

4. Vault returns temporary, short-lived credentials

5. Terraform deploys using the credentials returned by Vault and after a configurable amount of time, those temp credentials are deleted by Vault

## Terraform Registry

- A repository of publicly available Terraform providers and modules.


