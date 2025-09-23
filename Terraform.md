# Terraform


Terraform is an open-source Infrastructure as Code (IaC) tool by HashiCorp.

Lets you define, provision, and manage infrastructure using declarative config files written in HCL (HashiCorp Configuration Language).

Works with many providers: AWS, Azure, GCP, Kubernetes, VMware, etc.

Terraform engine ensures your infra matches the desired state by applying changes safely.

👉 In short: Terraform = IaC tool for automating cloud + on-prem infra with code.


## Why We Need Terraform

Without Terraform, infra is often managed manually (via AWS Console, CLI, etc.) or with provider-specific tools. This causes problems:

❌ Manual errors, inconsistent environments

❌ Hard to version-control infra

❌ Difficult to replicate infra across dev/stage/prod

❌ Poor visibility into changes

✅ Terraform solves these with:

* Infrastructure as Code (IaC) → infra defined in .tf files.

* Version control → track changes in Git.

* Automation → terraform apply provisions consistently.

* Idempotency → apply multiple times → same result.

* Multi-cloud & hybrid → one tool for many providers.

* Collaboration → state files + remote backends support team workflows.



### Terraform vs Ansible

| Feature                  | Terraform                   | Ansible                          |
|---------------------------|-----------------------------|----------------------------------|
| Purpose                  | Provision infrastructure    | Config management + app deployment |
| Language                 | Declarative (HCL)           | Imperative (YAML playbooks)      |
| Idempotency (infra state)| Built-in                    | Must be scripted carefully       |
| Multi-cloud infra        | ✅ Yes                      | ❌ Limited (uses cloud modules)  |

Terraform = infra creation. Ansible = config mgmt. They often work together.

### Terraform vs CloudFormation (AWS)


| Feature             | Terraform                          | CloudFormation         |
|----------------------|------------------------------------|------------------------|
| Language            | HCL                                | JSON/YAML             |
| Multi-cloud support | ✅ Yes                             | ❌ AWS only            |
| Modules/Reusability | Strong                             | Weaker                 |
| State mgmt          | Local/remote (S3, Consul, etc.)    | Managed by AWS         |
| Learning curve      | Easier (HCL)                       | Steeper (YAML/JSON)    |

👉 Terraform is more flexible & multi-cloud, CloudFormation is AWS-only but deeply integrated.





# Terraform Blocks

* A block is the main structural unit in Terraform.

* Starts with a type keyword (**resource, variable, provider**, etc.).

* Followed by optional labels (like resource type + name).

* Contains a body in { } with arguments.

👉 Think of blocks = sections, like objects in JSON/YAML.


```
resource "aws_instance" "my_ec2" {
  ami           = "ami-12345"
  instance_type = "t2.micro"
}
```

* **resource** = block type. it is the actuall block

* "aws_instance" = block label (resource type).(**parameters**)

* "my_ec2" = block label (resource name).

* Inside {} → arguments



## Types of Blocks

Some common block types:

1. Provider Block – configures a provider (AWS, Azure, GCP, etc.)
```
provider "aws" {
  region = "us-east-1"
}
```

2. Resource Block – defines infra resource.
```
resource "aws_s3_bucket" "my_bucket" {
  bucket = "my-bucket-name"
  acl    = "private"
}
```
3. Variable Block – input variables.
```
variable "instance_type" {
  type    = string
  default = "t2.micro"
}
```
4. Output Block – export values.
```
output "bucket_name" {
  value = aws_s3_bucket.my_bucket.id
}
```
5. Module Block – reuse Terraform configs.
```
module "vpc" {
  source = "./modules/vpc"
  cidr_block = "10.0.0.0/16"
}
```
6. Data Block – fetch read-only info from provider.
```
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
}
```




## Arguments vs Attributes

Arguments → values you define in config to tell Terraform how to create/manage something.

Attributes → values Terraform exposes after a resource is created, either from your arguments or from the provider’s API.

👉 Put simply:

Arguments = input (you write).

Attributes = output (Terraform gives back).


### Example
```
resource "aws_instance" "web" {
  ami           = "ami-12345"     # argument
  instance_type = "t2.micro"      # argument
}
```

After applying, Terraform knows more about this EC2 instance. You can reference its attributes:
```
output "instance_id" {
  value = aws_instance.web.id      # attribute
}

output "instance_public_ip" {
  value = aws_instance.web.public_ip   # attribute
}
```

# Terraform Arguments

* An argument is a key-value pair inside a block.

* Format: key = value.

* Define configuration for that block.

👉 Think of arguments = settings inside a block.


```
ami           = "ami-12345"   # argument
instance_type = "t2.micro"    # argument
```

Arguments can be:

* Literal values → strings, numbers, bools.

* References → variables, outputs, resources.

* Expressions → ${var.foo}, count.index.



# What is a Provider in Terraform?

* A provider is a plugin in Terraform that knows how to talk to an API (cloud provider, SaaS, on-prem system).

* Providers are responsible for creating, reading, updating, and deleting (CRUD) resources.

* Without a provider, Terraform wouldn’t know how to manage infrastructure.

👉 Example: aws, azurerm, google, kubernetes, helm, github.


```
provider "aws" {
  region  = "us-east-1"
  profile = "default"
}
```

* "aws" = provider name.

* Inside {} = arguments for authentication/config.

```
cd .terraform/providers/registry.terraform/hashicorp -> ls 

output : local, aws
```

**we can also provide more than same provider and can access it through alias.**

# What is String Interpolation?

* String interpolation = inserting variables, expressions, or resource attributes inside a string.

* In Terraform, this is done using ${ ... } syntax inside quotes.

- Example

```
variable "env" {
  default = "dev"
}

output "bucket_name" {
  value = "myapp-${var.env}-bucket"
}
```
# What are Provisioners?

* Provisioners in Terraform let you run scripts or commands on a resource after it is created (or before it is destroyed).

* Think of them as a “last resort” escape hatch when you need to do something Terraform itself can’t model directly.

👉 Example: Install software on an EC2 instance after Terraform creates it.



## Types of Provisioners
1. local-exec

    Runs a command on the machine running Terraform (your laptop, CI/CD agent).
    ```
    resource "aws_instance" "example" {
    ami           = "ami-12345"
    instance_type = "t2.micro"

      provisioner "local-exec" {
          command = "echo Instance ID: ${self.id} >> output.txt"
      }
    }
    ```
2. remote-exec

Runs commands on the created resource via SSH or WinRM.
```
resource "aws_instance" "example" {
  ami           = "ami-12345"
  instance_type = "t2.micro"

  provisioner "remote-exec" {
    inline = [
      "sudo apt update -y",
      "sudo apt install -y nginx"
    ]
  }

  connection {
    type        = "ssh"
    user        = "ubuntu"
    private_key = file("~/.ssh/id_rsa")
    host        = self.public_ip
  }
}
```

3. file
Uploads files/directories from local → remote machine.

```
provisioner "file" {
  source      = "app.conf"
  destination = "/etc/app/app.conf"
}
```


## When to Use Provisioners

✅ Use only when:

    A provider/resource doesn’t support what you need.

    Bootstrapping the machine (install packages, copy configs).

    Quick testing/demo setups.

⚠️ Best practice: Avoid heavy reliance on provisioners. Instead:

    Use configuration management tools (Ansible, Chef, Puppet).

    Use cloud-init / userdata for VMs.

    Use Terraform resources/providers directly.

## Why Provisioners are Considered “Last Resort”

  - They break Terraform’s idempotency model (runs may differ).

  - Harder to debug (commands may fail, leaving resources half-configured).

  - Not declarative → mixes procedural logic into IaC.

HashiCorp itself says:
👉 **“Provisioners should only be used as a last resort.”**




# Variables in Terraform

Variables are input values you can pass into Terraform configurations.

They make your configs dynamic, reusable, and configurable instead of hardcoding values.

👉 Think of them like function parameters in programming.


### You declare them using a variable block.
```
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}
```

* **description** → explains purpose.
* **type** → enforces data type (string, number, bool, list, map, object).
* **default** → optional fallback value.

### Using Variables

Use with the var. prefix.

```
resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type
}
```
## Types of variables


1) Input Variables

- Declared with variable blocks.

- Allow users to pass values into modules/config.

- Example:
```
variable "region" {
  type    = string
  default = "us-east-1"
}
```

- Used like:
```
provider "aws" {
  region = var.region
}
```

2) Output Variables

- Declared with output blocks.

- Show information after apply, or pass data between modules.

- Example:
```
output "instance_ip" {
  value = aws_instance.myvm.public_ip
}
```
3) Local Values (locals)

- Not user inputs, but computed constants inside the config.

- Example:
```
locals {
  app_name = "myapp"
  instance_tag = "web-${local.app_name}"
}
```
4) Environment Variables (TF_VAR_)

- Any environment variable prefixed with TF_VAR_ maps to an input variable.
```
Example:

export TF_VAR_region="us-west-2"
terraform apply
```

- Will override the default value of region.

## Variable Precedence in Terraform

Terraform has a well-defined hierarchy for variable values. If a variable is defined in multiple places, the higher precedence wins.

👉 Order of precedence (highest → lowest):

1. Explicit CLI flag
```
terraform apply -var="region=us-west-1"
```

✅ Always wins.

2. CLI var file flag
```
terraform apply -var-file="dev.tfvars"
```

3. Environment variables (TF_VAR_name)
```
export TF_VAR_region="us-east-2"
```

4. terraform.tfvars file (auto-loaded if present in working dir).

5. Any *.auto.tfvars file (auto-loaded, alphabetically).

6. Default value in variable block
```
variable "region" {
  default = "us-east-1"
}
```

👉 If no value is found from any of these → Terraform prompts you interactively at runtime.



# File types in terraform

1. Configuration Files (.tf)

    * Main files where you write infrastructure definitions.

    * Written in HCL (HashiCorp Configuration Language).

    * Examples:

        * main.tf → main resources.

        * variables.tf → variable definitions.

        * outputs.tf → outputs.

        * provider.tf → provider configs.

    👉 These are the files you usually commit to Git.


2. Variable Files (.tfvars / .tfvars.json)

    * Hold values for variables, separate from config.

    * Helps manage different environments (dev, staging, prod).

    Examples:

    **dev.tfvars**

    ```
    instance_type = "t2.micro"
    region        = "us-east-1"
    ```

3. State Files (terraform.tfstate, terraform.tfstate.backup)

    * Stores the current state of your infrastructure.

    * Maps your .tf configs to real resources (IDs, attributes, etc.).

    * terraform.tfstate → actual state.

    * terraform.tfstate.backup → previous state (before last apply).

    👉 Should be stored remotely (S3, GCS, Terraform Cloud) for team use.
    👉 Never edit manually unless absolutely necessary.


4. Lock File (.terraform.lock.hcl)

    * Tracks provider versions used in your project.

    * Ensures consistent versions across different machines/teams.

    * Created/updated automatically when you run terraform init.


# Looping in Terraform

1. count

    * Simplest way to create N copies of a resource.

    * Creates resources with indexes ([0], [1], …).

    ```
    resource "aws_instance" "servers" {
        count         = 3
        ami           = "ami-12345"
        instance_type = "t2.micro"
    }

    output "server_ids" {
        value = aws_instance.servers[*].id
    }
    ```

    ✅ Creates 3 EC2 instances.

2. for_each

    * Used when looping over maps or sets of strings.

    * Each resource gets a unique key.

    ```
    resource "aws_s3_bucket" "buckets" {
    for_each = toset(["logs", "images", "videos"])

    bucket = "myapp-${each.key}"
    }
    ```

    ✅ Creates 3 buckets → myapp-logs, myapp-images, myapp-videos.

    👉 Better than count when resources need unique names.

3. for expressions

    Create new lists or maps by transforming existing ones.

    Example 1: List transformation
    ```
    variable "zones" {
        default = ["us-east-1a", "us-east-1b", "us-east-1c"]
    }

    output "az_names" {
        value = [for z in var.zones : upper(z)]
    }
    ```
    ✅ Output → [ "US-EAST-1A", "US-EAST-1B", "US-EAST-1C" ]


    Example 2: Map transformation

    ```
    variable "instances" {
        default = {
            dev  = "t2.micro"
            prod = "t3.medium"
        }
    }

    output "instance_info" {
        value = { for name, type in var.instances : name => "Type: ${type}" }
    }
    ```
    ✅ Output →
    ```
    {
        dev  = "Type: t2.micro"
        prod = "Type: t3.medium"
    }
    ```

4. dynamic blocks

    * Used to loop over nested blocks (like security group rules).

    ```
    variable "ingress_ports" {
        default = [22, 80, 443]
    }

    resource "aws_security_group" "example" {
        name = "example-sg"

        dynamic "ingress" {
            for_each = var.ingress_ports
            content {
                from_port   = ingress.value
                to_port     = ingress.value
                protocol    = "tcp"
                cidr_blocks = ["0.0.0.0/0"]
            }
        }
    }
    ```
    ✅ Creates multiple ingress rules dynamically.



# State Management and remote backends

1. The Problem Before State (Why State Exists)

    Terraform is declarative: you write .tf configs (desired state). But Terraform also needs to know the current real-world state of your infra (what exists in AWS, GCP, etc.).

    👉 Without state:

        Terraform would have to query the cloud APIs for every plan/apply, which is slow & inconsistent.

        Terraform wouldn’t know if a resource was already created, deleted, or modified outside Terraform.

        Collaboration is impossible (different team members running Terraform would have different views).

    So Terraform introduced a state file (**terraform.tfstate**).


2. What is Terraform State?


A JSON file that Terraform uses to map resources in code → real resources in cloud.

Contains metadata (resource IDs, attributes, dependencies, outputs).


```
{
  "resources": [
    {
      "type": "aws_instance",
      "name": "web",
      "instances": [
        {
          "attributes": {
            "id": "i-0abcd1234",
            "ami": "ami-12345",
            "instance_type": "t2.micro",
            "public_ip": "54.123.45.67"
          }
        }
      ]
    }
  ]
}

```


3. Problems with Local State

By default, Terraform stores state in terraform.tfstate locally. Problems:

1. Collaboration Issues

    * If multiple engineers run Terraform → conflicts.

    * No way to lock state → race conditions → infra drift.

2. State File Corruption / Loss

    * If local file is lost, Terraform “forgets” infra.

    * Could accidentally recreate/delete resources.

3. Security Risks

    * State file contains sensitive info (passwords, secrets, tokens).

    * Keeping it on laptops is unsafe.


4. Backends (Solution)

A backend in Terraform determines:

    - Where state is stored.

    - How operations like plan & apply are executed.

Types:

1. Local Backend (default) → saves state as terraform.tfstate locally.

2. Remote Backends → store state in centralized locations, often with locking.



5. Remote Backends (Types & Benefits)
Common Remote Backends

    - AWS S3 + DynamoDB (locking)

    - GCP Cloud Storage + Firestore (locking)

    - Azure Blob Storage

    - Terraform Cloud / Enterprise

    - Consul, etcd, PostgreSQL

Benefits

✅ Centralized → one source of truth for team.
✅ Locking → prevents race conditions.
✅ Encryption → secure secrets.
✅ Versioning → recover from mistakes.
✅ Collaboration → multiple people/CI pipelines can run safely.




6. Example: S3 Backend with DynamoDB Locking

```
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```


- S3 → stores state file.

- DynamoDB → provides state locking (only one apply at a time).



7. State Management Commands


| Category        | Command Example                          | Description                         |
|-----------------|------------------------------------------|-------------------------------------|
| **Inspect State** | `terraform show`                        | Display the full state              |
|                 | `terraform state list`                   | List all resources in state         |
|                 | `terraform state show aws_instance.web`  | Show details of a specific resource |
| **Modify State** | `terraform state mv old_resource new_resource` | Move/rename a resource in state (rare, last resort) |
|                 | `terraform state rm aws_s3_bucket.old`  | Remove resource from state (rare)   |
| **Refresh State** | `terraform refresh`                     | Sync state with real infrastructure |





# Modules


1. What is a Module in Terraform?

    *  A module = a container for Terraform resources.

    * Every Terraform configuration is a module:

        * The folder where you run terraform apply = root module.

        * Any module called inside it = child module.

    👉 Think of modules like functions in programming — group related resources, make them reusable, and call them with different inputs.


2. Why Use Modules?

    ✅ Reusability → write once, use in multiple projects/environments.

    ✅ Organization → break big configs into smaller, logical units.

    ✅ Consistency → enforce standards across teams.

    ✅ Abstraction → hide complexity, expose only variables/outputs.


3. Types of Modules


1. Root Module → main folder where you run Terraform.

2. Child Module → referenced by the root (or another module).

3. Registry Module → published on **Terraform Registry**




4. Using a Module

You call a module with the module block:
```
module "ec2_instance" {
  source        = "./modules/ec2"
  instance_type = "t2.micro"
  ami           = "ami-12345"
}
```

- ```source``` → where module code lives:

    - Local (```./modules/ec2```)

    - Git repo (```git::https://github.com/org/repo.git```)

    - Terraform Registry (```terraform-aws-modules/vpc/aws```)



# Notes

1. there is no guarantee that indracture provising tool will update the reosurce implace, it can create a new resource also.








# Terraform init

terraform init

Purpose: initialize a working directory so Terraform can run there.

Main actions

1. Read terraform { required_providers { ... } } and module sources from your config.

2. Initialize backend:

    - If a remote backend is configured (S3, GCS, Terraform Cloud, etc.), init configures it and may prompt to migrate local state to the backend.

    - Creates ./terraform.tfstate remote mapping (or leaves it remote).

    - Supports -reconfigure to ignore any saved backend config and reinitialize.

3. Download provider plugins:

    - Downloads the provider binaries (e.g., hashicorp/aws) into .terraform/plugins (or provider cache).

    - Honours version constraints and the .terraform.lock.hcl lockfile (creates/updates lockfile).

    - You can pass -upgrade to fetch newer allowed provider versions.

4. Download modules:

    - Fetches any modules referenced via source = ... (registry, Git, local, HTTP) into .terraform/modules.

5. Create .terraform directory:

    - Stores plugin binaries, module code, and metadata.

6. Validate basic config:

    - Ensures provider requirements are satisfiable and modules can be fetched.

### Files created/updated

- ```.terraform/``` directory (providers, modules)

- ```.terraform.lock.hcl``` (locks provider checksums & versions)

- Backend-specific markers / local files about backend config

Flags & tips

terraform init -reconfigure — reinitialize backend ignoring any saved state.

terraform init -upgrade — upgrade provider versions to newest matching constraints.

If backend uses locking (e.g., S3+DynamoDB), init ensures locking resources exist/are reachable.




# terraform plan

Purpose: compute the execution plan — what Terraform would change to make real infrastructure match your configs.

- Run plan and save the plan (-out=planfile) if you want exact deterministic apply later.

### Main actions (in order)

1. Load configuration files (*.tf) and modules from .terraform/modules.

2. Load state:

    - Reads the current state from the configured backend (local file or remote).

3. Refresh current state (by default):

    - Terraform will query providers to read current live resource attributes and update the in-memory state to reflect the real world (this can be disabled with -refresh=false).

    - This step detects drift (manual changes outside Terraform) before making a plan.

4. Evaluate expressions / variables / interpolation:

    - Resolves variable values (from TF_VAR_, .tfvars, CLI -var, defaults), locals, and computed expressions.

5. Build dependency graph:

    - Reason about resource relationships (depends_on and implicit dependencies).

6. Compare desired config ↔ refreshed state:

    - Determine which resources will be create, update in-place, replace (destroy+create), or no-op.

7. Produce plan output:

    - Human-readable diff summary and resource actions.

    - If -out=planfile used — writes a binary planfile containing the planned actions.

8. Exit code:

    - 0 if plan succeeded (whether changes or no changes). Non-zero on error.

### Important details & flags

- ```-refresh=true|false``` — whether to refresh state before planning. Default is true (but you can disable).

- ```-out=plan.tfplan``` — write the plan to a file that you can feed to terraform apply.

    - Important: applying a saved plan uses the exact actions Terraform computed at plan-time (apply will not do another refresh if you pass the planfile).

- ```-var-file=FILE``` — load variable values from tfvars.

- ```terraform plan``` is read-only w.r.t provider APIs except for read calls during refresh.

- If the plan shows ```forces replacement``` it means some attribute changes cannot be done in-place.

Why save a plan?

- Deterministic apply:``` terraform apply plan.tfplan``` will execute exactly what was planned (no surprise if infrastructure changed after the plan). This is the recommended CI pattern.






# terraform apply

Purpose: actually make the changes needed to reach the desired state.

There are two common apply modes:

- terraform apply (no planfile): Terraform will run (implicit) plan steps (refresh, compute) and then prompt to apply.

- terraform apply plan.tfplan (saved plan): Terraform executes that saved plan without re-running refresh (so it's deterministic).

**Main actions when executing an apply (no planfile)**

1. Load config, state, refresh (by default) and build the plan (same as terraform plan).

2. Prompt for approval (unless -auto-approve given).

3. Acquire state lock (for remote backends with locking; prevents concurrent applies).

4. Execute resource operations:

    - Terraform computes the operation graph (order respecting dependencies).

    - It invokes provider plugin(s) to perform CRUD operations: create, update, delete on individual resources.

    - By default, Terraform performs operations in parallel (default parallelism = 10). Use -parallelism=N to change.

    - Each operation is executed and its result (attributes, IDs) returned by the provider plugin.

5. Update state incrementally:

    - After each resource operation completes, Terraform updates the in-memory state and writes it back to the backend (and keeps a backup copy of previous state, e.g., terraform.tfstate.backup if local).

    - Because it updates state incrementally, if apply fails mid-way you have a partial state that reflects completed changes.

6. Release lock (if any) once finished.

7. Compute outputs and show them.

8. Return exit code: 0 on success, non-zero on error.

### Main actions when executing an apply (using a planfile)

1. Verify planfile is valid for current configuration (some safety checks).

2. Acquire state lock.

3. Execute exact operations from planfile — no refresh is performed, so planfile must be recent and you must trust the plan.

4. Update state incrementally, release lock, show outputs.

### Provider invocation & plugins

1. Providers run as separate binary processes (plugin architecture) and Terraform communicates with them over RPC.

2. For each resource operation Terraform calls provider APIs (e.g., AWS, GCP) through the provider plugin.

3. Provider auth (ENV vars, credentials files) must be available to the process.

### State and locking

1. If your backend supports locking (e.g., S3 + DynamoDB table), Terraform will acquire a lock before modifying state and release it after. This prevents concurrent apply operations.

2. Terraform writes state to the backend after (or during) apply. A backup state (e.g., terraform.tfstate.backup) is normally kept.

### Parallelism and ordering

1. Terraform uses the dependency graph to determine which resources can run in parallel.

2. Default parallelism is 10; increase or decrease via -parallelism=N.

3. Some providers/resources have implicit/explicit dependencies causing serialization.

### Error handling & partial failures

1. If a resource operation fails, Terraform will stop (unless -replace/special handling).

2. You may need to fix the problem and run terraform apply again; state will reflect completed steps.

### Flags & useful options

- ```-auto-approve``` — skip interactive approval.

- ```-parallelism=N``` — number of concurrent resource operations.

- ``-target=resource`` — target a specific resource (advanced / not generally recommended).

- ``-refresh=false`` — when running apply without planfile, skip refresh first (dangerous).

- ``terraform apply -var-file=...`` — pass variables.

Extra: what Terraform state changes look like

- On create, provider returns the resource ID and attributes → Terraform records this in state.

- On update, Terraform records the new attributes.

- On destroy, Terraform removes resource entry from state.

- State holds mappings (resource address → real-world ID) and all attributes Terraform uses later.




# Data Block
- A data block in Terraform is used to fetch or reference existing information from providers instead of creating new resources.

- Think of it as read-only lookups.

- Useful when you want to use something already existing (like an AWS AMI, VPC, subnet, secret, etc.) in your configuration.

👉 Resource block = "create/manage something".
👉 Data block = "read/fetch something".


## General Syntax
```
data "<PROVIDER_TYPE>" "<NAME>" {
  # arguments / filters
}
```

- PROVIDER_TYPE → the type of data source (e.g., aws_ami, aws_vpc, azurerm_resource_group).

- NAME → local name you give it (used to reference later).

**Example 1: AWS AMI Lookup**

```
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}

resource "aws_instance" "example" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
}

```
- The data "aws_ami" "ubuntu" block fetches the latest Ubuntu AMI ID.

- aws_instance then uses this AMI without hardcoding it.



# Workspaces

1. What is a Workspace in Terraform?

- A workspace is a way to maintain multiple state files within the same Terraform configuration.

- By default, Terraform has one workspace: default.

- When you create new workspaces (dev, staging, prod), Terraform keeps a separate state file for each.

👉 Same configuration → multiple isolated states.

2. Why Workspaces?

Use cases:

- Manage multiple environments (dev, staging, prod) using the same code.

- Avoid duplicating Terraform config files per environment.

- Safely test changes in dev without touching prod.

3. How Workspaces Work (Internals)

- In local backend:

    - State files stored under:
    ```
    terraform.tfstate.d/<workspace_name>/terraform.tfstate
    ````

- In remote backend (e.g., S3, Terraform Cloud):

    - Terraform prefixes or namespaces state files per workspace automatically.

👉 Workspace selection only changes which state file Terraform reads/writes.
👉 Configuration files (.tf) stay the same.



4. Common Workspace Commands
```
# List workspaces
terraform workspace list
```
```
# Show current workspace
terraform workspace show
```
```
# Create a new workspace
terraform workspace new dev
```
```
# Switch to an existing workspace
terraform workspace select dev
```
```
# Delete a workspace (must not be active)
terraform workspace delete dev
```


# Depends on & null reosurce

1. depends_on in Terraform
What it is:

- An explicit dependency you can add to a resource or module.

- Normally, Terraform builds a dependency graph automatically (if resource A references resource B, Terraform knows A depends on B).

- But sometimes dependencies are implicit (e.g., order matters but no direct attribute reference). In those cases, you use depends_on.


```
resource "aws_instance" "app" {
  ami           = "ami-12345"
  instance_type = "t2.micro"

  depends_on = [aws_security_group.app_sg]
}
```
👉 Here, even if app doesn’t directly reference app_sg, Terraform ensures the security group is created first.


Use Cases:

✅ Force order of creation/destroy.
✅ Ensure networking is ready before compute.
✅ Ensure IAM role/policy exists before attaching.
✅ In modules:
```
module "app" {
  source = "./app"
  depends_on = [aws_vpc.main]
}
```




**2. null_resource in Terraform**\
What it is:

- A special resource that doesn’t create real infrastructure.

- Useful for running provisioners or triggers.

- Acts like a placeholder or hook in the graph.
```
Syntax:
resource "null_resource" "example" {
  provisioner "local-exec" {
    command = "echo Hello from Terraform"
  }
}
```

👉 Runs a local command on the machine running Terraform.

With ```triggers```:

- Triggers force recreation of a null_resource when inputs change.
```
resource "null_resource" "deploy" {
  triggers = {
    app_version = var.app_version
  }

  provisioner "local-exec" {
    command = "echo Deploying version ${self.triggers.app_version}"
  }
}
```

- If var.app_version changes, Terraform will destroy & recreate this resource.

Use Cases:

✅ Run local/remote scripts (bootstrap, config management).
✅ Execute commands during apply (e.g., ansible-playbook, kubectl apply).
✅ Use triggers to model changes when Terraform alone isn’t enough.
✅ Temporary glue between resources.

***3. depends_on + null_resource Together***

```
resource "null_resource" "wait_for_setup" {
  depends_on = [aws_instance.app]

  provisioner "remote-exec" {
    connection {
      type     = "ssh"
      host     = aws_instance.app.public_ip
      user     = "ec2-user"
      password = var.password
    }
    inline = [
      "echo App is ready",
      "sudo systemctl start myapp"
    ]
  }
}
```

👉 Here:

- Terraform ensures EC2 is up before running remote commands.

- The null_resource acts as a hook to configure the server after provisioning.

🔹 4. Best Practices & Warnings

- Use depends_on sparingly → prefer implicit dependencies (through references).

- Use null_resource only when there’s no provider alternative.

    - Example: Instead of null_resource + local-exec + aws CLI, prefer native aws_* resources.

- Overusing null_resource can make infra less declarative & reproducible.

5. Summary

- depends_on → explicitly control order of resources/modules.

- null_resource → placeholder resource to run provisioners or scripts, can be recreated with triggers.

- Together → useful for orchestration, waiting, or custom automation steps.
