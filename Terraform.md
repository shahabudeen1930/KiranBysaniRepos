## Terraform
it is used to provision, manage, automate infrastructure resources such as servers, databases, networks, and cloud resources (like AWS, Azure, GCP).

#### **Terraform File Structure**

```
terraform-project/
│── modules/                    # (Optional) Reusable Terraform modules
│   ├── networking/             # Configures networking resources like VPCs, subnets, route tables, and NAT gateways
│   │   ├── main.tf             # The primary Terraform configuration file - Defines infrastructure resources (e.g., VMs, networks, security groups).
│   │   ├── variables.tf        # Defines input variables - Declares input variables for modular and configurable deployments.
│   │   ├── outputs.tf          # Specifies output values - Specifies output values to display key resource attributes after deployment
│   ├── compute/
│       ├── main.tf
│       ├── variables.tf
│       ├── outputs.tf
│── environments/
│   ├── dev/                    # Environment-specific Terraform files
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── terraform.tfvars    # Stores the state of the infrastructure.
│   │   ├── backend.tf          # Specifies backend configuration for state storage - Defines remote state storage ( S3, Azure Blob)
│── main.tf                     # Root Terraform file (entry point)
│── variables.tf                # Input variables for Terraform configuration
│── outputs.tf                  # Output values
│── terraform.tfvars            # Define actual values for variables
│── backend.tf                  # 
│── provider.tf                 # Defines provider configurations - Configures cloud providers ( AWS, Azure) and required plugins.
│── README.md                   # Documentation
```
#### Terraform config details
Details
1. **Configuration Files (.tf and .tf.json)**
     - **main.tf** – The primary Terraform configuration file. - Defines infrastructure resources (e.g., VMs, networks, security groups).
     - **variables.tf** – Defines input variables. - Declares input variables for modular and configurable deployments.
     - **outputs.tf** – Specifies output values. -   Specifies output values to display key resource attributes after deployment.
     - **providers.tf** – Defines provider configurations - Configures cloud providers (e.g., AWS, Azure) and required plugins.
     - **terraform.tfvars** – Defines variable values (can also be named *.auto.tfvars).
     - **backend.tf** – Specifies backend configuration for state storage. - Defines remote state storage (e.g., S3, Azure Blob) for Terraform state management.
     - **user_data.sh** -  Contains initialization scripts for provisioning compute instances. bash or shell scripts.
2. **State Files**
     - **terraform.tfstate** – Stores the state of the infrastructure.
     - **terraform.tfstate.backup** – A backup of the previous state file.
3. **Lock Files**
     - **terraform.lock.hcl** – Ensures dependency consistency across Terraform runs.
4. **Module Files**
     - Any Terraform modules used within the project, usually stored in a modules/ directory.
5. **Provider and Plugin Files**
     - Located in .terraform/ & used to store provider plugins and downloaded modules.
     - Providers are plugins that allow Terraform to interact with various cloud platforms
6. **versions.tf** - Specifies Terraform and provider version constraints to ensure compatibility.
7. **locals.tf** - Defines reusable local variables for better readability and code reuse.
8. **iam_policies.tf** - Manages IAM roles, policies, and permissions for cloud resources.
9. **security_groups.tf** - Defines security groups and firewall rules for cloud networking
10. **network.tf** - Configures networking resources like VPCs, subnets, route tables, and NAT gateways.

#### tainted Flag in Terraform
- The tainted flag in Terraform indicates that a particular resource is in a "bad" or "inconsistent" state and needs to be recreated during the next terraform apply operation.
- This is useful when a resource is partially created or corrupted and needs a fresh deployment.
- **When to Use taint**
     - The resource is not functioning properly, but the infrastructure still recognizes it.
     - Manual intervention on the resource caused configuration drift.
     - Part of a testing scenario where you need to force recreation of specific resources.
       
#### Refresh
it will sync withthe state file, with the actual infrastructure. it only updates the state file, it does not modify infrastructure.

#### drift
command detects configuration drift by comparing the actual infrastructure state with the Terraform state. It helps identify unintended changes made outside of Terraform
- Causes of Drift:
     - Manual changes to infrastructure (like changing instance type from AWS Console).
     - Updates by external tools (like AWS CloudFormation, Ansible).
     - Misconfiguration in Terraform code.
- Fix Drift?
     - Option 1: Run terraform apply to sync infrastructure.
     - Option 2: Manually correct the infrastructure.
     - Option 3: Run terraform refresh to update the state without modifying infrastructure.

#### Backend - local and remote
backend in Terraform defines where the Terraform state file is stored
     - Local Backend   # Stores the state file locally on your machine (terraform.tfstate). { Best for individual developers }
     - Remote Backend  # Stores the state file remotely (like S3, Azure Blob, GCS). { Best for teams and collaboration }
- benifits
     - Collaboration (multiple users can work).
     - Disaster recovery (state file is backed up).

#### Terraform commands
**Terraform Command Workflow**
```terraform init                # Initializes the Terraform working directory. Downloads providers, modules, sets up the backend
terraform fmt                    # Formats the Terraform configuration files for readability
terraform validate               # Validates the syntax and structure of the Terraform configuration files
terraform plan                   # Shows the execution plan without applying changes.
terraform apply -auto-approve    # Applies the planned changes and creates/modifies resources
terraform state list             # Lists all resources in the Terraform state 
terraform destroy -auto-approve  # Destroys all resources defined in the configuration.
```

- **Terraform Initialization & Setup Commands**
   - **terraform init**		# Initializes the Terraform working directory. Downloads providers, modules, and sets up the backend.
   - **terraform version**	# Displays the Terraform version installed.
   - **terraform providers**	# Lists the providers required in the configuration.
   - **terraform fmt**		# Formats the Terraform configuration files for readability.
   - **terraform validate**	# Validates the syntax and structure of the Terraform configuration files.
- **Planning & Execution Commands**
   - **terraform plan**		# Shows the execution plan without applying changes. Identifies what will be created, updated, or destroyed.
   - **terraform apply**	# Applies the planned changes and creates/modifies resources.
   - **terraform destroy**	# Destroys the infrastructure managed by Terraform.
- **State Management Commands** - its a snapshot of the infrastructure's current configuration, used to manage and track changes efficiently.
   - **terraform state list**				# Lists all resources in the Terraform state.
   - **terraform state show <resource>**		# Displays detailed information about a specific resource.
   - **terraform state pull**				# Retrieves the current state file from the backend.
   - **terraform state push**				# Uploads a state file to the backend.
   - **terraform state rm <resource>**			# Removes a resource from the state file but does not delete it from the infrastructure.
   - **terraform state mv <source> <destination>**	# Moves a resource within the state file. Useful for renaming or restructuring resources.
- **Resource Management Commands**
   - **terraform import <resource> <id>**	# Imports an existing resource into Terraform state.
   - **terraform taint <resource>**		# Marks a resource for recreation during the next apply.terraform untaint <resource>	Removes the taint from a resource, preventing recreation.
   - **terraform refresh**			# Updates the state file with the actual infrastructure state (deprecated in newer versions).
- **Workspace & Environment Management**
   - **terraform workspace list**		# Lists all Terraform workspaces.
   - **terraform workspace show**		# Displays the current workspace.
   - **terraform workspace new <name>**		# Creates a new workspace.
   - **terraform workspace select <name>**	# Switches to a different workspace.
   - **terraform workspace delete <name>**	# Deletes a workspace (must not be in use).
- **Debugging & Logging**
   - **terraform output**	# Displays the output variables of the Terraform configuration.
   - **terraform debug**	# Enables debug logging for troubleshooting.
   - **terraform show**		# Displays the state or plan file in a human-readable format.
- **Terraform Modules & Dependencies**
   - **terraform get**		# Downloads and installs modules from the registry.
   - **terraform graph**	# Generates a dependency graph of the infrastructure.
- **Destroy & Clean Up**
   - **terraform destroy**			# Destroys all resources defined in the configuration.
   - **terraform apply -destroy**		# Equivalent to running terraform destroy.
   - **terraform workspace delete <name>**	# Deletes a workspace.

#### Statefile security
     - Use terraform login to store API credentials securely instead of hardcoding them.
     - Restrict state file access using IAM roles, ACLs, or RBAC policies.
     - Enable state locking (DynamoDB, Azure Locks, etc.) to prevent concurrent modifications.
     - Avoid storing state files in Git (use .gitignore to exclude terraform.tfstate).
     - Enable encryption (S3 SSE, Azure SSE, GCS KMS, Terraform Cloud).
     - Rotate secrets periodically to minimize risks if credentials are leaked.

#### Workspaces
- Terraform Workspaces allow you to manage multiple instances of the same infrastructure within a single backend.
- They help in separating environments (e.g., dev, staging, prod) without maintaining multiple Terraform configurations.

#### validate and plan
- terraform validate #  will  first check the catch syntax issues.
- terraform plan     # to preview changes before applying them.

#### Sample terraform files
Provision infrastructure using a declarative configuration language called HCL (HashiCorp Configuration Language). 
- It helps manage infrastructure across multiple cloud providers such as AWS, Azure, GCP, and on-premises environments.

#### Configuration Files
- Terraform uses .tf or .tf.json files to define infrastructure in HCL (HashiCorp Configuration Language). These files describe the desired state of the infrastructure.
```provider "aws" {
  region = "us-east-1"
}
resource "aws_instance" "example" {
  ami           = "ami-123456"
  instance_type = "t2.micro"
}
```
#### Providers
- Terraform interacts with various cloud services through providers. Each provider offers resources and data sources to manage infrastructure.
- AWS (provider "aws" { region = "us-east-1" })
- Azure (provider "azurerm" { features {} })
- Google Cloud (provider "google" { project = "my-project" })
- Kubernetes, GitHub, Docker, Helm, and many others.
```provider "aws" {
  region = "us-east-1"
}
```
#### Resources
- Resources define the infrastructure components such as VMs, networks, databases, etc.
- Example: Creating an AWS EC2 Instance
```resource "aws_instance" "example" {
  ami           = "ami-123456"
  instance_type = "t2.micro"
}
```

#### Data Sources
- Data sources allow Terraform to fetch information from existing infrastructure without modifying it.
- Example: Fetching an existing AWS AMI
```data "aws_ami" "latest" {
  most_recent = true
  owners      = ["amazon"]
}
```

#### Variables
- Variables make configurations dynamic and reusable.
- Example: Defining and Using Variable
```variable "instance_type" {
  default = "t2.micro"
}
resource "aws_instance" "example" {
  ami           = "ami-123456"
  instance_type = var.instance_type
}
```

#### Outputs
- Outputs display specific values from the Terraform execution.
- Example: Outputting the Public IP of an Instance
```output "instance_ip" {
  value = aws_instance.example.public_ip
}
```

#### State Management - Remote state
- Terraform maintains the current state of the infrastructure in a file called terraform.tfstate. This allows Terraform to understand what it manages.
- Remote State Example (AWS S3 Backend):
```terraform {
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "terraform.tfstate"
    region = "us-east-1"
  }
}
```
#### Modules
- Modules allow Terraform configurations to be reusable and modular.
- Example: Using a Module
```module "network" {
  source = "./modules/network"
}
```

#### components
Details
1. **Configuration Files (.tf and .tf.json)**
     - **main.tf** – The primary Terraform configuration file.
     - **variables.tf** – Defines input variables.
     - **outputs.tf** – Specifies output values.
     - **providers.tf** – Defines provider configurations (e.g., AWS, Azure, GCP).
     - **terraform.tfvars** – Defines variable values (can also be named *.auto.tfvars).
     - **backend.tf** – Specifies backend configuration for state storage.
2. **State Files**
     - **terraform.tfstate** – Stores the state of the infrastructure.
     - **terraform.tfstate.backup** – A backup of the previous state file.
3. **Lock Files**
     - **terraform.lock.hcl** – Ensures dependency consistency across Terraform runs.
4. **Module Files**
     - Any Terraform modules used within the project, usually stored in a modules/ directory.
5. **Provider and Plugin Files**
     - Located in .terraform/ and used to store provider plugins and downloaded modules.

#### Modules
Module is a container of multiple resources that are used together to manage infrastructure.
It is like a reusable component that can be called multiple times.
```module "network" {
  source = "./modules/network"
}
```

#### lock file 
(.terraform.lock.hcl) it ensures that the same provider version is used across different systems or environments. Prevents provider version conflicts.

#### sensitive data - passwords
inputs
     - Variables with sensitive = true.
     - Use Environment Variables.
     - Use Vault or Secrets Manager like AWS Secrets Manager

#### State file - important
it is a file that tracks of the infrastructure resources (terraform.tfstate)
     - Tracks the current state of infrastructure.
     - Maps resources in the real world to your configuration.
     - Helps Terraform know what changes need to be applied
     - Terraform uses state file to calculate changes.
     - Without state file, Terraform cannot manage existing infrastructure
- Example: If you have created an EC2 instance, Terraform will track its ID, IP, etc., in the state file.

#### State File deleted - recovery
state file (terraform.tfstate) is the most critical file in Terraform. If deleted, Terraform loses control over infrastructure
- Consequences of Losing the State File:
     - Terraform no longer knows what resources exist.
     - Any terraform apply can recreate resources, causing downtim
- Options to Recover the State File:
     - Retrieve from Remote Backend (Best Practice)
          - If you use a remote backend (S3, Azure Blob, GCS), retrieve the state file from there.
          - Example for S3 Backend:
     - Use terraform import
          - If no backup exists, import the infrastructure to recreate the state.
     - **Best Practice** - Always enable versioning in S3 or Git repository.

|Command	|Purpose|
|- |- |
|terraform state list	|List all resources in state |
|terraform state show <resource>	|Show details of a resource|
|terraform state mv <source> <destination>	|Move/rename a resource|
|terraform state rm <resource>	|Remove resource from state (without destroying)|
|terraform state replace-provider <old> <new>|	Change provider for all resources|
|terraform state pull	|Fetch state as JSON (backup)|
|terraform state push <file>	|Overwrite state with a new file|
|terraform refresh (deprecated)	|Sync state with infrastructure (use terraform apply -refresh-only)|
|terraform import <resource> <id>	|Import an existing resource into state|

#### Import
it allows to import existing infrastructure into Terraform state. It does not generate configuration files, only adds to the state.

#### replace 
Immediately destroys and recreates the resource without a plan.
     - taint requires a separate apply step.
     - replace does everything in one command.

#### Data sources
those are like, it allow to fetch data from existing infrastructure or external sources (like AWS, Azure) without modifying them.
     - You can reference existing infrastructure.
     - Avoid hardcoding values like AMI ID, VPC ID, etc.

#### Sentinel
Sentinel is a policy as code framework which helps enforce policies on infrastructure
- use cases
     - Prevent creating expensive EC2 instances.
     - Block public access to S3 buckets.
     - Enforce encryption policies

#### Graph 
graph command shows the dependency graph of all resources. It shows how resources are connected (dependencies).

####  deploy CI/CD (Jenkins)
Pipeline Stages:
     - Checkout Code: Clone Git repo.
     - Terraform Init: Initialize terraform.
     - Terraform Plan: Check infrastructure changes.
     - Terraform Apply: Deploy infrastructure.

#### State Locking 
state locking - it prevents multiple team members from modifying infrastructure simultaneously.
- Example:
     - Developer A runs terraform apply → Terraform locks the state.
     - Developer B tries to apply → Error: "State is locked"
- State Locking Works
     - Local Backend: No state locking.
     - Remote Backend (S3, Azure, GCS): Automatically locks the state.

#### Circular Dependencies
Circular dependencies occur when two resources depend on each other
- Fix Circular Dependencies
     - Use depends_on attribute.
     - Break the dependency chain.

#### Blue-Green Deployment and Canary Deployments
 Answer:
- Blue-Green Deployment (Zero Downtime)
     - Blue Environment: Current production.
     - Green Environment: New environment.
     - After testing, switch traffic to Green
- Canary Deployment
     - Deploy to 5% users.
     - Monitor errors.
     - Slowly increase traffic.
     - Use Terraform with Traffic Manager (ALB, Route53) for canary.

#### Provisioners
Provisioners are used to execute local-exec or remote-exec commands during resource creation or destruction.
- Best Practice: Provisioners should be used as a last resort for tasks that Terraform itself cannot handle.

#### Error: provider configuration not present
Solutions for this error:
     - Ensure terraform init is run before applying the configuration.
     - Check that the provider block is properly defined.
     - Verify provider versions in the .terraform.lock.hcl file.
- This prevents downtime by ensuring the new instance is ready before terminating the old one.

#### manage zero-downtime deployment
Techniques for Zero-Downtime Deployment:
     - Use Load Balancers to route traffic to healthy instances.
     - Use create_before_destroy in Terraform for controlled replacement.
     - Implement a Blue-Green Deployment strategy.

#### rollback Terraform changes after a failed deployment?
Terraform doesn't provide an automatic rollback, but you can manually revert changes by:
     - Step 1: Use terraform destroy to delete the incorrect resources.
     - Step 2: Checkout the previous working commit from your version control system.
     - Step 3: Run terraform apply to recreate resources in their original state.

#### integrate with CI/CD pipelines 
CI/CD pipeline for Terraform follows these steps:
     - Lint Code: Use tflint or checkov for static analysis.
     - Plan: Run terraform plan for previewing changes.
     - Apply: Run terraform apply to deploy changes in non-prod environments.
     - Manual Approval for Prod: Ensure human intervention for production deployments.

#### test code before deploying infrastructure?
Best practices for testing Terraform code:
     - terraform validate → Validates syntax and configuration.
     - terraform plan → Previews infrastructure changes.
     - Use checkov, tflint, or terrascan for static code analysis.
     - Use kitchen-terraform for integration testing in CI/CD pipelines.


#### terraform init initializes a Terraform working directory. It:
- terraform init initializes a Terraform working directory. It:
- Downloads the necessary provider plugins.
- Sets up the backend for state storage.
- Prepares the directory for further Terraform commands.

#### Terraform Provider
- A provider is a plugin that allows Terraform to interact with APIs (e.g., AWS, Azure, GCP). It defines what and how Terraform manages resources.

#### terraform validate
- This checks the syntax and configuration of Terraform files to ensure they are valid

#### terraform plan
- It shows what Terraform will do before applying changes. It creates an execution plan so you can review and approve changes safely.

#### terraform apply 
- This applies the changes required to reach the desired state defined in your configuration.

#### terraform destroy
- It destroys all resources managed by Terraform in the current working directory.

#### State File Locking
- It prevents concurrent operations on the same state file, which could lead to corruption or inconsistent state. Especially useful in team environments.

#### Remote Backend
- A backend defines where state is stored. A remote backend (like S3 or Azure Blob) allows multiple team members to share the same state securely.

#### Tainted Resource
- A resource marked as "tainted" will be destroyed and recreated on the next apply. It's used when something has gone wrong or is out of sync.

#### Terraform D
- terraform.d is a hidden directory where Terraform **stores local plugin binaries and other internal data**.

#### Expression Evaluation
- It refers to how Terraform evaluates variables, conditionals, functions, etc., in .tf files during configuration parsing.

#### Vertex Evaluation
- Each configuration block (resource, module, variable, etc.) is a vertex in Terraform’s internal dependency graph. Vertex Evaluation is the step where Terraform resolves dependencies and evaluates those blocks.

#### Graph Walk
- It’s the process Terraform uses to traverse its dependency graph to determine the order of resource creation, updates, or deletion.

#### Graph Builder
- It builds the internal dependency graph by analyzing resource relationships and expressions.

#### State Manager
- The component responsible for reading, writing, locking, and managing the Terraform state file.

#### Configuration Loader
- It loads and parses .tf configuration files into a format Terraform can understand.

#### CLI
- CLI stands for Command Line Interface. It's the main way users interact with Terraform by running commands like init, plan, apply, etc.

#### Backend
- A backend in Terraform determines how state is loaded and how operations like plan and apply are executed.

#### Resource Graph
- It’s a visual or logical representation of all resources and their dependencies in a configuration. Helps Terraform know execution order.

#### Terragrunt - use cases
- Terragrunt is a wrapper for Terraform that:
     - Helps with managing modules.
     - Supports remote state backends and dependencies better.
- uses cases Terragrunt is a wrapper for Terraform that helps:
     - Manage complex infrastructure with many modules.
     - DRY up repetitive code.
     - Handle remote state and environment configuration.
     - Run dependencies in the right order.

#### Terraform Core
- Terraform Core is the engine of Terraform. It:
     - Parses configuration.
     - Builds dependency graphs.
     - Creates and executes plans.
     - Interacts with providers to manage resources.

#### Terraform State
- it is a file that tracks resources Terraform manages. It keeps information like resource IDs and dependencies to map real infrastructure to your config.

#### terraform fmt
- terraform fmt formats Terraform code to follow standard style conventions, improving readability and consistency.

#### Terraform benfits
- Infrastructure as Code (IaC) with version control.
- Multi-cloud support.
- Immutable infrastructure.
- Automation of provisioning and teardown.
- Strong community and modularity.

#### lock Terraform module versions
- Use version constraints in the module source:
```
module "example" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 3.0"
}

```

- se Git tag/commit in source URL if pulling from Git.

#### Modules
- A module is a container for multiple resources that are used together. You can create reusable infrastructure by writing your own modules or using public ones
- Modules are containers of Terraform configuration grouped to manage multiple resources together. They promote reuse and maintainability.

#### built-in provisioners - remote-exec - local-exec
- file – Uploads files to remote machines.
- remote-exec – Executes commands on remote machines.
- local-exec – Executes commands on the machine running Terraform.

#### components of Terraform architecture
- Core – Handles graph building, state management, and planning.
- Providers – Interact with APIs (e.g., AWS, Azure).
- State File – Stores infrastructure state.
- Configuration – User-defined HCL files.
- Provisioners – Optional step for post-creation actions.

#### count
- The count meta-argument lets you create multiple instances of a resource:
```
resource "aws_instance" "example" {
  count = 3
  ami = "ami-123456"
  instance_type = "t2.micro"
}
```

#### Duplicate Resource" error
- Ensure unique resource names in your .tf files.
- Avoid re-defining a resource already declared.
- Manage state carefully when moving or copying code.

#### ignore duplicate resource error during terraform apply
- Terraform does not allow ignoring this error by default.
- Best practices:
     - Import existing resources with terraform import.
     - Use terraform state rm to remove stale resource references.
     - Avoid defining the same resource more than once.

#### cloning an infrastructure
- Use modules for reusability.
- Parameterize via variables.
- Use remote backends to avoid state issues.
- Separate environments (dev/stage/prod) using workspaces or directories.

#### null_resource
- A null_resource allows running provisioners without creating any real infrastructure:
```
resource "null_resource" "example" {
  provisioner "local-exec" {
    command = "echo Hello"
  }
}
```

#### Azure callbacks
- Azure Event Grid + Azure Functions to trigger Terraform runs.
- Used in automation pipelines or CI/CD flows for post-deployment actions.

#### Terraform work
- Loads config files → Builds dependency graph → Creates plan → Applies changes using provider APIs → Updates state.

#### Terraform plugins
- During terraform init, Terraform scans the config and:
     - Downloads required providers (plugins).
     - Caches them in .terraform/plugins or Terraform registry.

#### upgrade plugins
- **terraform init -upgrade**
- This updates provider plugins and modules to the latest acceptable versions based on constraints.

#### object of one module available to another module
- Use outputs from one module and pass them as inputs to another:
```
module "network" {
  source = "./network"
}

module "compute" {
  source   = "./compute"
  subnet_id = module.network.subnet_id
}
```

#### control and handle rollbacks when something goes wrong
- Terraform has no built-in rollback. You can:
     - Use versioned backups of the state file.
     - Roll back manually by applying a previous version.
     - Use VCS + CI/CD to track and revert config changes.

#### Recover from a failed apply
- Analyze the error message.
- Fix the issue in the config or environment.
- Re-run terraform apply.
- If stuck, you may need to use terraform state commands or taint resources.

#### bulk import the state of current cloud subscription
- Not directly. But tools like Terraformer or custom scripts can auto-generate .tf files and import resources:
- **terraformer import aws --resources=vpc,subnet,ec2**

#### features
- Infrastructure as Code (IaC)
- Multi-cloud support
- Dependency resolution (graph-based)
- Plan & Apply workflow
- Modules & Reusability
- Remote state management

#### encounter a serious error and want to rollback
- Terraform doesn’t support automatic rollback. You can:
     - Restore a previous state backup.
     - Revert configuration code.
     - Manually destroy/replace resources if needed.

#### Sub-graphs
- Sub-graphs are smaller pieces of the full resource dependency graph. Terraform evaluates these sub-graphs in parallel when there are no dependencies between them.

#### terraform.tfstate
- It stores the current state of the infrastructure. Terraform uses this to determine what actions to perform during plan or apply.

#### Local-exec and remote-exec
- local-exec: Executes commands on your local machine.
- remote-exec: Executes commands on the remote machine (via SSH or WinRM).

#### tainted resource
- A tainted resource is marked for destruction and recreation during the next apply, even if its configuration hasn’t changed.

#### variables supported
- String
- Number
- Bool
- List
- Map
- Object
- Tuple
- Set

#### Data Source
- Data sources let Terraform fetch information from outside without creating new resources.
- Use cases:
     - Get existing VPC IDs, AMIs, user data, etc.
     - Reference shared infrastructure

#### Terraform Registry
- A public repository for:
     - Providers
     - Modules - URL: registry.terraform.io

#### Random
- Used to generate random values:
     - Passwords
     - Pet names
     - IDs - Useful for resource uniqueness or entropy.

#### implicit and explicit dependencies
- Implicit: Based on references (e.g., resource_a.id in resource_b)
- Explicit: Using depends_on to force a dependency

- depends_on = [aws_instance.web]

#### rename an existing resource
- Terraform tracks resources using resource names in the state file. To rename a resource without destroying it:
     - Step 1: Move the state manually using terraform state mv
          - terraform state mv aws_instance.old_name aws_instance.new_name
     - Step 2: Update your .tf code with the new name.
          - This avoids deletion and recreation of the resource.
