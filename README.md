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

- Hashicorp recommends using provisioners as a last resourt, giving preference to try using inherent mechanism wihtin your infrastructure deployment to carry out custom tasks where possible.
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
