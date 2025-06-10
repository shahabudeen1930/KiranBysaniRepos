#### End-to-End Cloud Architecture
Design a cloud-native platform for a multi-tier web application (include compute, storage, networking, security).
- 3-tier web app on AWS with :
     - Frontend (React app) hosted on S3 with CloudFront for CDN and HTTPS.
     - Backend (Node.js microservices) deployed on EKS (Kubernetes) with Ingress via AWS ALB Controller.
     - Database layer using Amazon RDS (PostgreSQL) in Multi-AZ.
     - Networking: VPC with public/private subnets, NAT Gateways for outbound traffic, and Route53 for DNS.
     - Security: IAM roles per service, KMS for encryption, WAF + Shield for DDoS protection, and SSO via AWS IAM Identity Center.
     - Used Terraform for IaC and ArgoCD for GitOps-based deployment.

#### cloud-native microservices
- Security was layered:
     - Network: Used private subnets, NACLs, and security groups. Service-to-service traffic in EKS was encrypted with mTLS using Istio.
     - IAM: Services used IRSA (IAM Roles for Service Accounts) for scoped permissions.
     - Secrets: Managed using AWS Secrets Manager or HashiCorp Vault.
     - CI/CD: Scanned images with Trivy during pipeline, enforced no critical vulnerabilities before pushing to ECR.
     - Logging and Alerts: Integrated AWS CloudTrail, GuardDuty, and Security Hub for continuous monitoring.

#### observability stack
- We used:
     - Prometheus + Grafana for metrics and dashboards
     - Loki for logs
     - Jaeger or X-Ray for tracing
     - Alertmanager for notifications (Slack/email)
- I chose these for their open-source flexibility, Kubernetes integration, and cost-effectiveness. For enterprise setups, we also integrated Datadog or New Relic where centralized dashboards and smart alerting were needed.

#### High availability - HA & Disaster recovery - DR
- I design for HA by:
     - Deploying across multi-AZ (and multi-region when needed)
     - Using autoscaling groups, managed services (RDS Multi-AZ, ALB, S3), and self-healing Kubernetes workloads
- For DR:
     - Automated snapshot backups of DBs and EBS
     - Used infrastructure versioning with Terraform for rapid rebuilds
     - Created runbooks and tested failover to a secondary region using DNS routing (e.g., Route53 failover policy)

#### On-Premises - Cloud Migration - challanges
- I led the migration of a monolithic Java app from on-prem VMs to AWS:
     - Used CloudEndure to rehost legacy apps
     - Refactored some services into containers (ECS Fargate)
     - Migrated databases from Oracle to Aurora PostgreSQL with DMS
     - Set up hybrid VPN and AD Federation
- Challenges:
     - Legacy dependencies and hardcoded IPs slowed containerization
     - Initial VPN throughput was a bottleneck—resolved by switching to AWS Direct Connect
     - Lift-and-shift workloads required performance tuning post-migration

#### stateful workloads during migration
We used snapshot replication or DMS with CDC (change data capture) for minimal downtime.
- For example:
     - Migrated MySQL using AWS DMS with CDC, cut over during a low-traffic window
     - File servers were synced with rsync + EFS
          - We tested migration in staging environments and validated data integrity before switching DNS.

#### SLOs SLIs
- For a public API:
    - SLIs: latency < 200ms, error rate < 0.1%, availability > 99.9%
    - SLOs: 99.95% availability over 30 days
    - We collect metrics via Prometheus, set thresholds in Alertmanager, and review SLO burn rates using Grafana.
    - Regular SRE review meetings discuss error budget usage and remediation actions.

#### incident management process.
- We followed a blameless incident response model:
    - Detect: Alert triggered by monitoring
    - Triage: Use runbooks to identify scope
    - Mitigate: Rollback or failover
    - Postmortem: RCA with action items and timeline
- Used PagerDuty for alerting, StatusPage for comms, created Jira tasks for remediation.

#### version upgrades without downtime
- details
    - Used blue-green or canary deployments via Argo Rollouts
    - Kubernetes workloads upgraded using rolling updates with readiness probes
    - For stateful services like DBs, performed in-place upgrades in staging, then used read replica promotion for live cutover
    - Backed up everything before upgrade and tested rollback steps

#### Terraform File Structure
- we use terraform to integrate into CI/CD pipelines to support infrastructure automation, version control, deployments across different environments like prod, non-prod, dev, test.
- Key Use Cases:
     - **Provisioning Resources**: We use Terraform to create VMs, Virtual Networks, NSGs, Public IPs, Azure Kubernetes Service (AKS), Storage Accounts, etc.
     - **Modular Design**: We've built reusable modules for networking, compute, storage.
     - **State Management**: prevent state conflicts, We use remote state stored in Azure Storage (Blob + SAS Token).
     - **Environment Isolation**: Different workspaces (dev, staging, prod) help us isolate infrastructure across environments using the same codebase.
     - **Security & Governance**: Sensitive data like secrets or keys Insted of hardcoding we use Terraform variables and environment variables.
 
```
terraform-project/
│── modules/                    # (Optional) Reusable Terraform modules
│   ├── networking/
│   │   ├── main.tf		# Root Terraform file (entry point)
│   │   ├── variables.tf	# Input variables for Terraform configuration
│   │   ├── outputs.tf		# Output values
│   ├── compute/
│       ├── main.tf
│       ├── variables.tf
│       ├── outputs.tf
│── environments/
│   ├── dev/                    # Environment-specific Terraform files
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── terraform.tfvars	# Define actual values for variables
│   │   ├── backend.tf		# Remote state configuration
│── main.tf                     
│── variables.tf                
│── outputs.tf                   
│── terraform.tfvars            
│── backend.tf                   
│── provider.tf                   # Cloud provider configurations
│── README.md                     # Documentation
```
#### Terraform use cases
- Infra provisioning and genral use case
     - We use Terraform to provision and manage all our cloud infrastructure on Azure. This includes VMs, networks, firewalls, and sometimes AKS clusters.
     - We store state remotely in Azure Blob Storage and use workspaces for environment separation (dev/stage/prod).
     - Terraform is integrated into our CI/CD pipeline in Azure DevOps — every infra change goes through a PR + plan + apply workflow.
- AKS focused
     - In my project, Terraform is used to provision and manage AKS clusters and supporting resources like Azure VNETs, subnets, load balancers, and ACR.
     - Once AKS is provisioned, ArgoCD or Helm takes over for app-level deployments.
     - Using Terraform helped us create reproducible AKS environments for dev, staging, and production.
- Security/Key Vault
     - We use Terraform to provision Azure infrastructure including Key Vaults, VMs, and Storage.
     - Secrets and keys are stored securely in Azure Key Vault, and we use Terraform data sources to retrieve them dynamically during provisioning.
     - This helps ensure secrets are never hardcoded or exposed in code or pipelines.

#### Terraform config details
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
3. **Lock Files** - **terraform.lock.hcl** – Ensures dependency consistency across Terraform runs.
4. **Module Files** - Any Terraform modules used within the project, usually stored in a modules/ directory.
5. **Provider and Plugin Files** - Located in .terraform/ and used to store provider plugins and downloaded modules.
6. **versions.tf** - Specifies Terraform and provider version constraints to ensure compatibility.
7. **locals.tf** - Defines reusable local variables for better readability and code reuse.
8. **iam_policies.tf** - Manages IAM roles, policies, and permissions for cloud resources.
9. **security_groups.tf** - Defines security groups and firewall rules for cloud networking
10. **network.tf** - Configures networking resources like VPCs, subnets, route tables, and NAT gateways.

#### Terraform taint
- Terraform taint just marks one resource as “dirty” so Terraform will destroy and recreate just that one in the next apply.
- Tainted Flag : 
     - The tainted flag in Terraform indicates that a particular resource is in a "bad" or "inconsistent" state and needs to be recreated during the next terraform apply operation. This is useful when a resource is partially created or corrupted and needs a fresh deployment.
- **When to Use taint**
     - The resource is not functioning properly, but the infrastructure still recognizes it.
     - Manual intervention on the resource caused configuration drift.
     - Part of a testing scenario where you need to force recreation of specific resources.
       
#### Terraform drift
- command detects configuration drift by comparing the actual infrastructure state with the Terraform state. It helps identify unintended changes made outside of Terraform, ensuring consistency and compliance.

#### Terraform lifecycle rules
- lifecycle rules help control how Terraform treats a resource.
- We can use:
     - **create_before_destroy**: to avoid downtime—it creates the new resource first, then deletes the old one.
     - **prevent_destroy**: to protect important resources, like production DBs.
     - **ignore_changes**: if we don’t want Terraform to freak out when some values are changed manually.

#### Terraform Command Workflow
- Terraform commands

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

#### Terraform Statefile / lock / deleted
- it's not good to keep the state file locally, haaa… Instead, we use a remote backend, like Amazon S3, Terraform Cloud, or Azure Blob Storage, so everyone uses the same state.
- Locking
     - We enable state locking using something like DynamoDB in AWS to avoid two people updating at the same time.
- unclock
     - Make sure no one else is actively running Terraform commands.
     - If you or someone else closed the terminal or killed Terraform mid-command, the lock might not have been released properly.
     - Make 100% sure no one else is currently running Terraform commands before you delete the lock
          - Delete that lock file manually **.terraform.tfstate.lock.info**
          - or we can force unlock it **terraform force-unlock <LOCK_ID>**
- if **Statefile deleted** accidentally 
- Was it local or remote
     - Local (terraform.tfstate): Check if you have a backup or version control.
     - Remote (e.g., S3, Terraform Cloud): Most backends (S3, GCS, etc.) support versioning. You might be able to roll back.
- Recover from Backup/Versioning (if possible)
- Local backups: Terraform often creates **terraform.tfstate.backup**. Use this if it's recent.
     - cp terraform.tfstate.backup terraform.tfstate
- Remote state with versioning (S3, GCS, etc.):
     - S3: Go to the S3 console > Bucket > Object versions > Restore previous version.
     - Terraform Cloud: Check under "States" in the workspace for a history.
- If No Backup Is Available — Recreate the State
     - Option A: Use terraform import - Manually import each resource back into the state:
          - terraform import aws_instance.example i-0abcd1234efgh5678
     - Option B: Terraform Refresh (less accurate) 

#### Refactor
- s the process of improving the structure, organization, maintainability of Terraform code without changing the actual infrastructure it manages.
- It's similar to code refactoring in software development — the goal is to clean up, modularize, make your Terraform configurations easier to work with, scale, collaborate on.

1. Your code becomes large and hard to manage:
     - All resources are defined in a single .tf file or folder, making collaboration and debugging difficult.
2. You're working with multiple environments (dev/staging/prod):
     - Hardcoded values and duplicate code create risk. Refactoring helps introduce variable files, modules, and workspaces.
3. You need reusable, DRY code:
     - Instead of copying-pasting similar blocks for every resource, refactoring allows you to create modules and reuse them cleanly.
4. You want better CI/CD integration:
     - Modular code is easier to validate, test, and promote safely through environments using pipelines.
5. You want to enforce security and consistency:
     - Refactor to remove hardcoded secrets, enforce naming conventions, tagging policies, etc.

- Refactoring Activities Include:
     - Splitting large .tf files by resource type or function.
     - Creating reusable Terraform modules.
     - Using variables and *.tfvars for environment-specific values.
     - Switching to a remote backend (like Azure Storage) for state management.
     - Removing hardcoded secrets (use Azure Key Vault or environment variables).
     - Adding comments, naming conventions, tags, and version control for providers.

#### **Statefile security**
- info
     - Use terraform login to store API credentials securely instead of hardcoding them.
     - Restrict state file access using IAM roles, ACLs, or RBAC policies.
     - Enable state locking (DynamoDB, Azure Locks, etc.) to prevent concurrent modifications.
     - Avoid storing state files in Git (use .gitignore to exclude terraform.tfstate).
     - Enable encryption (S3 SSE, Azure SSE, GCS KMS, Terraform Cloud).
     - Rotate secrets periodically to minimize risks if credentials are leaked.

#### Multiple servers Terraform count
- **count** - if your resources are all the same, use count.
     - count is like using a number. So you say count = 5 and it makes 5 copies.
- **for_each** - It's more flexible and better when you want to give unique names or values to each resource..
     - for_each is used when you want to loop over a list or a map. 

#### Terrraform Workspaces
- Terraform Workspaces allow you to manage multiple instances of the same infrastructure within a single backend. They help in separating environments (e.g., dev, staging, prod) without maintaining multiple Terraform configurations.
- some teams prefer to use different state files and directory structures instead of workspaces—depends on the use case.

#### Terraform validate - plan
     - terraform validate #  will  first check the catch syntax issues.
     - terraform plan     # to preview changes before applying them.

#### Terraform Secrets 
- We usually avoid putting secrets directly in .tf files or tfvars.
- Instead, we can:
     - Use tools like Vault, SSM Parameter Store, or Secrets Manager
     - Or pull secrets dynamically using data sources , Also, we keep the state file encrypted, and if it’s remote like in S3, we make sure it has bucket encryption and limited access.

#### Structure Terraform code
- for big projects, we break things into modules.
- Like, one module for network, one for compute, one for databases. Then we create a main root module that calls all the others. This makes the code clean, reusable, and easier to maintain.
- Also, we keep environment-specific variables in separate files and sometimes even use tools like **Terragrunt** to help manage multiple environments.

#### Terragrunt
- Terragrunt is basically a wrapper around Terraform—it doesn’t replace Terraform, it just makes managing big Terraform projects easier.
- When you're dealing with multiple environments, modules, and state files, Terraform alone starts to get messy.
- Terragrunt helps by:
     - Reducing code duplication
     - Managing remote backends automatically
     - And making it easier to do multi-environment setups
- Example :
     - so in one of my projects, we had to manage infra for dev, stage, and prod, across multiple teams.
     - We used Terraform modules, but each team had its own set of variables, backends, and region configs.
     - So we used Terragrunt folders, Each folder had a terragrunt.hcl file that:
          - Called the right Terraform module
          - Passed inputs
          - Configured the remote state backend (like S3 or Azure Blob)
          - Set dependency on other modules (like compute depends on network)
     - we could deploy just one environment at a time using terragrunt apply-all or even a single component
- Challange, one challenge was managing dependencies between modules.
     - Like, if network wasn’t ready, compute would fail. Terragrunt has a dependency block, but sometimes the outputs didn’t load right because of timing.
     - Also, debugging errors is a bit harder with Terragrunt because it’s layering on top of Terraform, so the logs can get noisy.
	
#### Terraform Module
- A Terraform module is a reusable collection of Terraform configuration files that groups related resources together. Instead of writing the same code repeatedly, you create a module and reuse it.
- Think of a Terraform module as a function in programming—it encapsulates logic that can be used multiple times with different inputs.

- Use Modules
     - Code Reusability – Write once, use multiple times
     - Simplification – Organizes and structures Terraform configurations
     - Scalability – Easily manage complex infrastructure
     - Maintainability – Easier to update and manage
     - Standardization – Enforce best practices across multiple projects
- Types
     - Root Module – The main Terraform configuration in the working directory
     - Child Modules – Separate module directories that are called from the root module
     - Published Modules – Modules from the Terraform Registry (e.g., Terraform AWS/Azure Modules)

#### Terraform import existing infrastructure
- if you already have a resource that wasn’t created by Terraform, you can use terraform import.
     - terraform import aws_instance.myec2 i-1234567890abcdef
- you still have to manually write the code for that resource. Import just maps the existing thing into the state—it doesn’t generate .tf code.

#### Terraform Remote Backend
- A remote backend in Terraform allows you to store your Terraform state file (terraform.tfstate) in a remote location instead of locally.
- This is important for:
     - Collaboration – Multiple team members can work on infrastructure simultaneously
     - State Locking – Prevents multiple users from applying changes at the same time
     - Security – Protects sensitive data (like passwords & keys)
     - Automation – Allows CI/CD pipelines to manage infrastructure
- multiple remote backends
     - Azure Storage Account (for Terraform on Azure)
     - AWS S3 with DynamoDB (for Terraform on AWS)
     - Terraform Cloud
- Remote Backend with AWS S3 -  If you’re using AWS, configure S3 & DynamoDB for remote state:
```
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-lock"
    encrypt        = true
  }
}
```

#### Import AWS Resources into Terraform

1. Identify AWS resource ID (EC2, S3, etc.)
2. Define a Terraform resource block (main.tf)
3. Use terraform import to bring it into state
4. Run terraform state show to get resource details
5. Update main.tf with actual attributes
6. Run terraform plan and terraform apply

- Manually Identify the AWS Resource
     - Find the AWS resource ID (e.g., instance ID for EC2, ARN for S3, etc.)
     - You can get this from the AWS Console or by using AWS CLI:
     - EC2
          - aws ec2 describe-instances --query "Reservations[].Instances[].InstanceId"
     - S3 Bucket
          - aws s3 ls
     - Security Groups
          - aws ec2 describe-security-groups --query "SecurityGroups[].GroupId"

- Define the Terraform Configuration - Before importing, create the Terraform resource block without terraform apply.
- Example: EC2 Instance (main.tf)
```
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "my_ec2" {
  # The attributes will be populated after the import
}
```
- Import the Resource into Terraform - Now, import the AWS resource into Terraform state using the terraform import command.
- For EC2 Instance
     - terraform import aws_instance.my_ec2 i-0abcd1234efgh5678
- For S3 Bucket
     - terraform import aws_s3_bucket.my_bucket my-existing-bucket-name
- For Security Group
     - terraform import aws_security_group.my_sg sg-0123456789abcdef0

- Generate the Terraform Configuration
- Once the resource is imported, Terraform knows about it, but it doesn't write the configuration in main.tf.
     - To generate it, run
          -  terraform plan -generate-config-out=generated.tf
     - OR manually check the state
          - terraform state show aws_instance.my_ec2
- Validate and Apply - After updating main.tf, run:
     - terraform plan & terraform apply

#### Terraform and observability tools to improve reliability
- IaC (Terraform) ensures:
    - Consistency across environments
    - Fast recovery using terraform apply
    - Version control and reviewable changes
- Observability:
    - Tracked error spikes, slow queries, pod OOMs
    - Alerted on anomalies and set up auto-scaling policies based on metrics
    - Combined, these reduce manual errors and accelerate MTTR (mean time to recovery).


#### infrastructure as code in Azure
- info
    - Terraform as the primary IaC tool for cross-cloud portability and module reusability
    - Azure Bicep for native ARM deployments when working in Azure-only environments
    - All IaC code is versioned in Azure Repos (or GitHub) with Azure Pipelines enforcing PR-based deployments and security scanning (using tools like Checkov or tfsec)
    - Used Terraform backend with Azure Storage and state locking with Azure Blob leases

#### Azure automation to improve reliability
- We had a critical app running in Azure App Services that experienced occasional memory leaks.
 - Log Analytics alerts on high memory usage
 - An Azure Automation Runbook that auto-scaled the app plan or recycled the app pool
 - Logged actions and sent Slack alerts via Logic App
 - This reduced downtime incidents by 80% and removed the need for manual intervention during off-hours.

#### CircleCI
CircleCI is a cloud-native CI/CD platform that automates code testing, building, and deployment. It supports integration with GitHub, Bitbucket, GitLab, and can run on both cloud and self-hosted runners.
- Core Features:
     - YAML-based workflows
     - Parallelism & caching
     - Docker-native builds
     - Integration with Kubernetes, AWS, GCP, Azure
     - Reusable executors, orbs (reusable packages)
     - Insights & metrics dashboard
- Key Concepts
     - **.circleci/config.yml** -	Defines the pipeline structure
     - **Jobs** -	A collection of steps (scripts, commands)
     - **Workflows** -	Orchestrate jobs, define dependencies
     - **Executors** -	Environment to run jobs (docker, machine, macos)
     - **Orbs** -	Reusable CircleCI packages (prebuilt jobs/commands)
     - **Contexts** -	Inject environment-specific secrets and configs 

- Orbs are reusable CircleCI packages containing jobs, commands, and executors. They're great for:
     - Reusing CI logic (e.g., Docker image publishing, Slack notifications)
     - Integrating 3rd-party tools (e.g., Datadog, Terraform)


#### Kafka Architecture
- Apache Kafka is a distributed event streaming platform designed for high throughput, scalability, and fault tolerance.
- It follows a publish-subscribe model and is widely used for real-time data streaming, event-driven architectures, log processing.

#### Producers
     - Producers write (publish) data to Kafka topics.
     - They push messages to a broker, which assigns them to a partition.
     - Key Features:
          - Asynchronous processing – Reduces latency.
          - Partitioning logic – Messages can be partitioned by key.
          - Acks configuration for reliability (acks=0, acks=1, acks=all).

#### Kafka Broker (Server)
     - A broker is a Kafka server responsible for storing, receiving, and serving messages.
     - Kafka clusters consist of multiple brokers.
     - Each broker handles multiple partitions of different topics.
     - Key Features:
          - Scalability – New brokers can be added dynamically.
          - Fault Tolerance – Replication ensures data durability.
          - Leader Election – One broker is assigned as the partition leader; others are followers.

#### Partitions & Replication
     - Kafka splits topics into partitions to achieve parallelism.
     - Each partition is stored in multiple brokers using replication (replication.factor).
     - Leader-Follower Model:
          - Leader partition handles all read/writes.
          - Followers replicate data and take over if the leader fails.
     - ISR (In-Sync Replicas):
          - Brokers that have fully replicated the leader’s data.
          - Messages are committed only when written to min.insync.replicas.

#### Consumers & Consumer Groups
     - Consumers read messages from Kafka topics.
     - Each consumer belongs to a consumer group.
     - Kafka ensures each partition is consumed by only one consumer within a group.
     - Key Concepts:
          - Consumer Offset Management: Kafka keeps track of the last read message using offsets (stored in Kafka or an external store like Zookeeper).
          - Rebalancing: If a consumer joins/leaves a group, Kafka reassigns partitions.
          - Parallel Processing: Multiple consumers can process different partitions simultaneously.

#### Zookeeper (Metadata Management)
     - Kafka (before version 2.8) relies on Zookeeper for:
          - Leader election (chooses partition leaders).
          - Broker discovery & failure detection.
          - Topic and partition metadata storage.
          - Consumer group coordination.


#### monitoring and observability in DevOps
- info
     - Metrics: Prometheus, Grafana, AWS CloudWatch, Datadog.
     - Logging: ELK Stack (Elasticsearch, Logstash, Kibana), Loki.
     - Tracing: OpenTelemetry, Jaeger for distributed tracing.
     - Alerting: PagerDuty, AlertManager, OpsGenie.

1. Infrastructure Metrics
     - CPU Usage (%)
     - Memory Usage (%)
     - Disk I/O (Read/Write Speed, Latency)
     - Disk Usage (%)
     - Network Bandwidth (Inbound/Outbound Traffic)
     - Network Packet Loss & ErrorsSystem Load Average
2. Kubernetes Metrics
     - Pod CPU & Memory Usage
     - Pod Restart Count
     - Container Uptime & Status
     - Node Resource Utilization
     - Kube API Server Latency & Request Rate
     - Kubernetes Scheduler Performance
     - Kubernetes etcd Health & Latency
3. Application Performance Metrics
     - Request Latency (p50, p95, p99 percentiles)
     - Request Throughput (Requests per Second - RPS)
     - HTTP Status Codes (2xx, 4xx, 5xx errors)
     - Database Query Latency
     - Cache Hit Ratio
     - Thread/Connection Pool Utilization
4. Database Metrics
     - Query Execution Time
     - Active Connections
     - Replication Lag
     - Cache Miss Ratio
     - Database Disk Usage
     - Transaction Commit/Rollback Count
5. Security Metrics
     - Unauthorized Access Attempts
     - IAM Policy Changes
     - SSL/TLS Certificate Expiry
     - API Gateway Unauthorized Errors
     - Unusual Login Patterns (Anomaly Detection)
6. Network & API Metrics
     - API Request Success & Failure Rate
     - DNS Resolution Time
     - Load Balancer Connection Count
     - Rate of 4xx and 5xx Errors
     - TCP Retransmissions & Packet Drops
7. Log-Based Metrics
     - Error Rate (Number of ERROR Logs/Min)
     - Application Crash Frequency
     - Slow Query Logs Count
     - Critical System Events (Service Restart, Failures)
8. Business & SLA Metrics
     - Uptime (SLA Compliance %)
     - SLO (Service Level Objective) Violations
     - Customer Transactions Per Second
     - Revenue Impacting Errors



#### Kubernetes YAML files 

- **Application Deployment Files**
     - **deployment.yaml** -	Define stateless application deployments
     - **statefulset.yaml** -	Define stateful applications (e.g., DBs)
     - **daemonset.yaml** -	Run a pod on all (or some) nodes
     - **job.yaml** -	Run a one-time batch job
     - **cronjob.yaml** -	Run scheduled jobs (like cron tasks)
     - **replicaset.yaml** -	Manage identical pod replicas (usually via Deployment)
     - **pod.yaml** -	Define an individual pod (basic use/testing)

- **Networking & Access**
     - **service.yaml** -	Expose internal/external access to pods
     - **ingress.yaml** -	Define HTTP/HTTPS routing to services
     - **networkpolicy.yaml** -	Control network communication between pods
     - **endpoint.yaml** -	Manually define network endpoints

- **Configuration & Secrets**
     - **configmap.yaml** -	Store non-sensitive config data
     - **secret.yaml** -	Store sensitive data like passwords, tokens
     - **env-config.yaml** -	Environment variables for applications

- **Storage Configuration**
     - **persistentvolume.yaml** -	Define physical storage
     - **persistentvolumeclaim.yaml** -	Request for persistent storage
     - **storageclass.yaml** -	Define dynamic storage provisioning
     - **volume.yaml** -	Define ephemeral or mounted volume
     - **volumeclaimtemplate.yaml** -	Template for dynamic PVCs in StatefulSets

- **Security & Access Control**
     - **serviceaccount.yaml** -	Define a Kubernetes Service Account
     - **role.yaml** -	Namespace-level permissions
     - **clusterrole.yaml** -	Cluster-wide permissions
     - **rolebinding.yaml** -	Bind Role to subjects (users/groups/SAs)
     - **clusterrolebinding.yaml** -	Bind ClusterRole to subjects
     - **podsecuritypolicy.yaml** -	(Deprecated) Pod-level security controls
     - **securitycontext.yaml** -	Pod/container-level security context

- **Scaling & Monitoring**
     - **horizontalpodautoscaler.yaml** -	Auto-scale pods based on CPU/memory
     - **metrics-server.yaml** -	Install resource metrics for HPA
     - **custom-metrics.yaml** -	Extend metrics for autoscaling

- **Lifecycle, Init, Probes**
     - **init-container.yaml** -	Define init containers for a pod
     - **readiness-probe.yaml** -	Define readiness checks for a container
     - **liveness-probe.yaml** -	Define health checks for a container
     - **startup-probe.yaml** -	Define startup checks for slow-starting apps

- **testing & Debugging**
     - **debug-pod.yaml** -	Launch debug containers
     - **ephemeral-container.yaml** -	Temporarily inject containers for debugging

- Helm/Templating (If Used)
     - **values.yaml** -	Define default values for Helm chart templates
     - **templates/deployment.yaml** -	Helm deployment template
     - **Chart.yaml** -	Chart metadata

####  3-tier Java web application Github
- Inside Each Folder
     - **base/** -	Global configs, ingress rules, secrets, SA, and policies
     - **frontend/** -	Frontend deployment and HPA (React/Angular/Vue)
     - **backend/** -	Java Spring Boot app with deployment, service, autoscaler
     - **database/** -	Stateful DB (e.g., PostgreSQL or MySQL) with PVC
     - **monitoring/** -	Prometheus & Grafana for metrics and visualization
     - **helm-chart/** -	Optional Helm chart for templated deployment
     - **scripts/** -	Bash script to apply the entire YAML structure

```
sample-java-k8s-app/
├── README.md
├── manifests/
│   ├── base/
│   │   ├── namespace.yaml
│   │   ├── configmap.yaml
│   │   ├── secret.yaml
│   │   ├── serviceaccount.yaml
│   │   ├── ingress.yaml
│   │   ├── networkpolicy.yaml
│   │   └── storageclass.yaml
│   ├── backend/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── hpa.yaml
│   ├── frontend/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── hpa.yaml
│   ├── database/
│   │   ├── statefulset.yaml
│   │   ├── service.yaml
│   │   ├── persistentvolume.yaml
│   │   └── persistentvolumeclaim.yaml
│   └── monitoring/
│       ├── prometheus-config.yaml
│       ├── prometheus-deployment.yaml
│       ├── grafana-deployment.yaml
│       ├── grafana-service.yaml
│       └── service-monitor.yaml
├── helm-chart/ (optional, if using Helm)
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│       └── (all above resources as templates)
└── scripts/
    └── deploy.sh
```

- deploy.sh
  
```
#!/bin/bash
echo "Applying Kubernetes YAML manifests..."
kubectl apply -R -f manifests/base
kubectl apply -R -f manifests/database
kubectl apply -R -f manifests/backend
kubectl apply -R -f manifests/frontend
kubectl apply -R -f manifests/monitoring
echo "Deployment initiated!"
```

#### LDAP
- LDAP is a protocol used to access and manage directory services.

- Microsoft Active Directory and OpenLDAP implement LDAP
     - AD is a full enterprise directory with Kerberos, GPO, and rich integration with Windows environments.
     - OpenLDAP is a flexible, open-source LDAP server that's great for Linux systems and custom deployments

- LDAP is Used
     - Centralized Authentication – Log in once, access many systems (SSO).
     - Role-based Access Control (RBAC) – Assign roles based on LDAP groups.
     - User Provisioning – Auto-create accounts across systems using LDAP data.
     - CI/CD & Monitoring Tools Integration – Jenkins, GitLab, Grafana, etc.

- LDAP Security Best Practices I Follow:
1. Centralized Authentication via LDAP
     - Integrate OpenLDAP or Active Directory (AD) with Jenkins, GitLab, Kubernetes, and internal apps.
     - Use LDAP groups to control access levels via RBAC (e.g., dev, admin, read-only roles).
2. Secure LDAP (LDAPS)
     - Always use LDAPS (LDAP over SSL/TLS) to encrypt credentials in transit.
     - Enforce TLS 1.2+ and use trusted CA certificates for LDAP clients.
3. Bind DN with Least Privilege
     - Use a dedicated bind user with limited access to perform directory lookups.
     - Avoid using admin/root bind accounts in services or pipelines.
4. Account Policies and Expiry
     - Enforce password policies, account lockout, and expiry rules on LDAP accounts.
     - Automate inactive account cleanup via cron jobs or directory monitoring.
5. Access Auditing
     - Enable LDAP query logging and integrate with SIEM tools (e.g., Splunk, ELK) to monitor access attempts and anomalies.
6. LDAP Integration in CI/CD & Platforms
     - Secure Jenkins, GitLab, and Kubernetes dashboards via LDAP auth.
     - Use LDAP groups to dynamically control access to resources in automation scripts or Terraform modules.
7. Just-In-Time (JIT) Access
     - Combine LDAP with Just-in-Time (JIT) provisioning for ephemeral access (e.g., for SREs needing emergency access).

#### Security Best Practices

1. Identity & Access Management (IAM)
     - Enforce principle of least privilege (PoLP) for users, roles, and service accounts.
     - Use role-based access control (RBAC) for Kubernetes, cloud IAM, and Git platforms.
     - Enable MFA (multi-factor authentication) for critical services and admin access.
2. Secrets Management
     - Never hardcode credentials or secrets in code or pipelines.
     - Use tools like HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, or Kubernetes secrets with encryption at rest.
     - Rotate secrets and tokens regularly with automation.
3. CI/CD Security
     - Use signed commits and verify artifact integrity.
     - Run static (SAST) and dynamic (DAST) security scans using tools like SonarQube, Trivy, Checkov, or AquaSec.
     - Enforce pre-commit hooks, policy checks (OPA), and security gates in pipelines.
4. Infrastructure as Code (IaC) Hardening
     - Scan IaC templates (Terraform, ARM, CloudFormation) for misconfigurations.
     - Apply least privilege for cloud resources, disable public access by default.
     - Use tools like Checkov, TFSec, and Terraform Sentinel for policy compliance.
5. Container Security
     - Use minimal base images (e.g., distroless, Alpine).
     - Scan container images for vulnerabilities before pushing (Trivy, Anchore).
     - Sign and verify Docker images (Cosign, Notary).
     - Use runtime protection tools like Falco or AppArmor.
6. Kubernetes Security
     - Implement network policies to restrict pod communication.
     - Enforce pod security standards (PSPs/PSS).
     - Isolate workloads with namespaces, limit privileges (no root containers).
     - Use tools like Kube-bench, Kube-hunter, OPA/Gatekeeper.
7. Network Security
     - Use firewalls, security groups, NSGs, and WAFs to control traffic.
     - Encrypt data in transit with TLS 1.2+, use HTTPS for all endpoints.
     - Restrict SSH/RDP access using bastion hosts or Session Manager.
8. Monitoring & Alerting
     - Integrate security events into centralized logging (ELK, Splunk, or CloudTrail).
     - Set alerts for unauthorized access attempts, port scans, or misconfig changes.
     - Use SIEM integrations for incident detection and response.
9. Patch Management
     - Automate patching for OS, dependencies, and containers.
     - Use managed services wherever possible to offload patch responsibility.
10. Compliance & Auditing
     - Use tools like AWS Config, Azure Policy, and Terraform Sentinel for compliance.
     - Enable audit logging for cloud and Kubernetes environments.

#### EC2 Instance Suddenly Stops Responding

- Problem: A critical EC2 instance running in production suddenly stops responding to requests. Pinging the instance shows no response, and SSH access fails.
- Solution:
     1. **Check AWS Console & CloudWatch Logs:** Verify if the instance is running and check system logs.
     2. **Verify Security Groups & NACLs:** Ensure inbound/outbound rules allow necessary traffic.
     3. **Check CPU & Memory Utilization:** If high, try restarting the instance.
     4. **Check EBS Volume Health:** If the root volume is corrupted, detach it, attach it to another instance, and repair it.
     5. **Recreate the Instance:** If nothing works, create a new instance from a backup or AMI.

#### Data Loss Due to S3 Bucket Misconfiguration

- Problem: A developer accidentally deletes important objects from an S3 bucket that had no versioning enabled.
- Solution:
     1. **Restore from Backups:** If AWS Backup or replication is enabled, restore from the latest backup.
     2. **Enable Versioning & MFA Delete:** Prevent accidental deletions in the future.
     3. **Use S3 Object Lock:** Enable retention policies for critical objects.
     4. **Configure IAM Policies:** Implement least privilege access and enable logging using AWS CloudTrail.

#### High Latency in RDS Database Queries

- Problem: An Amazon RDS database suddenly experiences high latency, affecting application performance.
- Solution:
     1. **Check CloudWatch Metrics:** Look for high CPU, memory, or IOPS utilization.
     2. **Enable Performance Insights:** Identify slow queries and optimize them using indexing or query refactoring.
     3. **Increase Instance Size:** Upgrade to a larger instance type if the load is high.
     4. **Use Read Replicas & Caching:** Implement RDS Read Replicas and Redis/Memcached for better performance.
     5. **Review Connection Pooling:** Ensure efficient use of database connections.

#### Auto Scaling Group Fails to Launch New Instances

- Problem: An Auto Scaling Group (ASG) is set up to scale automatically, but it fails to launch new instances when demand increases.
- Solution:
     1. **Check IAM Role Permissions:** Ensure ASG has permissions to launch EC2 instances.
     2. **Verify Launch Template/Configuration:** Ensure it references the correct AMI, instance type, and security groups.
     3. **Check Quotas & Limits:** AWS account limits may prevent new instances from being created.
     4. **Check AZ Availability:** If a specific AZ is out of capacity, allow multi-AZ deployment.
     5. **Inspect Health Checks:** If ELB health checks fail, adjust thresholds or instance warm-up times.

#### Lambda Function Execution Timeout

- Problem: A Lambda function fails due to timeout issues, causing incomplete execution.
- Solution:
     1. **Increase Timeout Limit:** Default is 3 seconds; extend it as needed (max 15 minutes).
     2. **Optimize Code Execution:** Reduce response times by optimizing loops, API calls, and database queries.
     3. **Use Asynchronous Invocation:** If processing is long, switch to SQS or SNS for async execution.
     4. **Enable Provisioned Concurrency:** Reduce cold start delays for latency-sensitive functions.
     5. **Monitor with CloudWatch Logs:** Identify bottlenecks using AWS X-Ray and CloudWatch

#### Automated CI/CD Pipeline for a Microservices Application

- Tech Stack:
     - Cloud: AWS (EKS, S3, IAM, CodeBuild, CodePipeline)
     - CI/CD Tools: GitHub Actions, AWS CodePipeline
     - Containerization: Docker
     - Orchestration: Kubernetes (EKS)
     - Monitoring & Logging: Prometheus & Grafana

- Pipeline Workflow:
     1. Developer Pushes Code → Code is pushed to GitHub repository.
     2. CI Phase (Build & Test)
          - GitHub Actions triggers on push.
          - Runs linting, unit tests using Jest (Node.js) or PyTest (Python).
          - Builds a Docker image and pushes it to AWS ECR.
     3. CD Phase (Deploy to Kubernetes)
          - AWS CodePipeline pulls the latest image.
          - Helm charts deploy it to Amazon EKS.
          - Kubernetes applies rolling updates.
     4. Post-Deployment Steps
          - Monitored using Prometheus and Grafana.
          - Logs stored in AWS CloudWatch.
- Key Features to Highlight in Interview:
     - End-to-End Automation – No manual steps required after a commit.
     - Rolling Updates in Kubernetes – Ensures zero downtime deployment.
     - Security – IAM roles, secrets management, vulnerability scanning.
     - Monitoring & Logging – Real-time visibility into the application.

#### GitHub Actions YAML
- (.github/workflows/cicd-pipeline.yml)
     - Triggers on push to main
     - Runs tests, builds Docker image, pushes to AWS ECR, deploys to EKS
     - Rolling update for zero-downtime deployment

```
name: CI/CD Pipeline with GitHub Actions
on:
  push:
    branches:
      - main
jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install Dependencies
        run: npm install

      - name: Run Tests
        run: npm test

      - name: Build Docker Image
        run: docker build -t my-app:${{ github.sha }} .
  push-to-ecr:
    needs: build-and-test
    runs-on: ubuntu-latest
    steps:
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Login to Amazon ECR
        run: aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com

      - name: Push Image to ECR
        run: |
          docker tag my-app:${{ github.sha }} <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/my-app:${{ github.sha }}
          docker push <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/my-app:${{ github.sha }}
  deploy-to-eks:
    needs: push-to-ecr
    runs-on: ubuntu-latest
    steps:
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v1

      - name: Update Kubeconfig
        run: aws eks update-kubeconfig --region us-east-1 --name my-cluster

      - name: Deploy to EKS
        run: |
          kubectl set image deployment/my-app my-app=<AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/my-app:${{ github.sha }}
          kubectl rollout status deployment/my-app
```

#### GitLab CI/CD YAML
- (.gitlab-ci.yml)
     - Runs in GitLab CI/CD pipeline
     - Builds & pushes Docker image to AWS ECR
     - Deploys to Kubernetes using Helm

```
stages:
  - test
  - build
  - deploy
variables:
  AWS_REGION: "us-east-1"
  ECR_URL: "<AWS_ACCOUNT_ID>.dkr.ecr.$AWS_REGION.amazonaws.com"
  IMAGE_TAG: "$CI_COMMIT_SHORT_SHA"
test:
  stage: test
  image: node:18
  script:
    - npm install
    - npm test
build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t $ECR_URL/my-app:$IMAGE_TAG .
    - echo $AWS_SECRET_ACCESS_KEY | docker login -u AWS --password-stdin $ECR_URL
    - docker push $ECR_URL/my-app:$IMAGE_TAG
deploy:
  stage: deploy
  image: bitnami/kubectl
  script:
    - aws eks update-kubeconfig --region $AWS_REGION --name my-cluster
    - helm upgrade --install my-app ./helm-chart --set image.repository=$ECR_URL/my-app --set image.tag=$IMAGE_TAG
```

#### Jenkins Pipeline
- (Jenkinsfile)
     - Runs a multistage pipeline
     - Uses Jenkins agents for testing, building, and deploying

```
pipeline {
    agent any
    environment {
        AWS_REGION = 'us-east-1'
        ECR_URL = '<AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/user/repo.git'
            }
        }
        stage('Run Tests') {
            steps {
                sh 'npm install && npm test'
            }
        }
        stage('Build and Push Docker Image') {
            steps {
                sh """
                    docker build -t $ECR_URL/my-app:$BUILD_NUMBER .
                    aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_URL
                    docker push $ECR_URL/my-app:$BUILD_NUMBER
                """
            }
        }
        stage('Deploy to EKS') {
            steps {
                sh """
                    aws eks update-kubeconfig --region $AWS_REGION --name my-cluster
                    kubectl set image deployment/my-app my-app=$ECR_URL/my-app:$BUILD_NUMBER
                    kubectl rollout status deployment/my-app
                """
            }
        }
    }
}
```

#### CodePipeline
- pipeline.yml)
     - Automates build and deployment using AWS CodePipeline
 
```
version: 0.2
phases:
  install:
    runtime-versions:
      nodejs: 18
  pre_build:
    commands:
      - npm install
      - npm test
  build:
    commands:
      - docker build -t $AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/my-app:$CODEBUILD_RESOLVED_SOURCE_VERSION .
      - docker push $AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/my-app:$CODEBUILD_RESOLVED_SOURCE_VERSION
  post_build:
    commands:
      - aws eks update-kubeconfig --region us-east-1 --name my-cluster
      - kubectl set image deployment/my-app my-app=$AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/my-app:$CODEBUILD_RESOLVED_SOURCE_VERSION
```

#### Azurepipeline
- azure-pipelines.yml
     - Runs in Azure DevOps pipeline
     - Deploys to Azure Kubernetes Service (AKS)

```
trigger:
  branches:
    include:
      - main
pool:
  vmImage: 'ubuntu-latest'
stages:
- stage: Build
  jobs:
  - job: Build
    steps:
    - task: NodeTool@0
      inputs:
        versionSpec: '18.x'
    - script: |
        npm install
        npm test
    - task: Docker@2
      inputs:
        command: buildAndPush
        containerRegistry: 'myDockerRegistry'
        repository: 'my-app'
        tags: '$(Build.BuildId)'
- stage: Deploy
  jobs:
  - job: Deploy
    steps:
    - script: |
        az aks get-credentials --resource-group myResourceGroup --name myAksCluster
        kubectl set image deployment/my-app my-app=myDockerRegistry.azurecr.io/my-app:$(Build.BuildId)
```

### Step by step CI/CD Jenkins Pipeline from scratch 
- code commit to production deployment. when new code is tested, integrated, and deployed
- work flow
     - Code Commit → 2. Build → 3. Test → 4. Security Scanning → 5. Package → 6. Deploy → 7. Monitor
     - Rollback (if needed)

- **Environments** Used
     - **Development**: Local environment where developers write and test code.
     - **Testing/Staging**: Pre-production environments where integration and performance tests run.
     - **Production**: Live environment where the final application is deployed.

1. Stage 1: Code Commit & Version Control (CI Start)
     - Developers push code to GitHub/GitLab/Bitbucket.
     - Each commit triggers the pipeline automatically using webhooks.
     - Branching Strategy (Git Flow, Trunk-Based, etc.) determines the workflow.
     - Tools Used: **Git, GitHub/GitLab/Bitbucket**

2. Stage 2: Code Quality & Static Analysis
     - Runs Linting, Code Formatting, and Static Code Analysis to catch syntax and styling issues early.
     - Tools like SonarQube, ESLint, Pylint, or Checkstyle are used.
     - Tools Used: **SonarQube**, ESLint, Pylint, Flake8, Checkstyle*

3. Stage 3: Build & Compile
     - Compiles the application (if necessary) and packages it.
     - Ensures dependencies are installed correctly.
     - If using Docker, builds a Docker Image.
     - Tools Used: **Maven (Java), Gradle, NPM (JavaScript), Docker**

4. Stage 4: Unit Testing
     - Runs unit tests to check if individual components function correctly.
     - Ensures that small, isolated parts of the application work as expected.
     - Tools Used: **JUnit (Java), PyTest (Python)**, Jest (JavaScript), Mocha

5. Stage 5: Integration Testing
     - Tests interactions between different modules and services.
     - Runs API and contract testing.
     - Tools Used: **Postman, Selenium**, Newman, Cypress, Pact
  
6. Stage 6: Security Scanning (Optional but Recommended)
     - Scans code for vulnerabilities using security tools.
     - Checks for secrets in code and outdated dependencies.
     - Tools Used: **Trivy, Snyk**
  
7. Stage 7: Artifact Creation & Storage
     - Packages the application into a deployable artifact:
     - WAR/JAR (Java)
     - Docker Image
     - Binary (Go, C, Rust)
     - Stores it in artifact repositories like:
     - DockerHub, JFrog Artifactory, Nexus, AWS ECR
     - Tools Used: **Docker, Nexus, JFrog Artifactory**
  
8. Stage 8: Deployment to Staging/Testing
     - Deploys the application to a staging environment.
     - Runs end-to-end testing and performance tests.
     - Validates that the build works in a production-like setting.
     - Tools Used: **Kubernetes, Terraform, Ansible, Helm, AWS ECS**
  
9. Stage 9: Approval & Manual Testing (Optional)
     - If a manual approval step is needed (e.g., compliance/security team review), it happens here.
     - QA Team may perform exploratory testing.
     - Tools Used: **Jira, Slack, GitHub Actions** Manual Approval
  
10. Stage 10: Production Deployment (CD Start)
     - Deploys the tested and approved application to production.
     - Deployment strategies:
          - Blue-Green Deployment (zero downtime)
          - Canary Release (gradual rollout)
          - Rolling Deployment (gradual replacement)
     - Uses Infrastructure as Code (IaC) tools for deployment automation.
     - Tools Used: **Kubernetes, Helm, Terraform, Ansible**
   
11. Stage 11: Post-Deployment Monitoring
     - Logs & Metrics Collection (Observability)
     - Monitors performance, availability, and error rates.
     - Alerts teams if issues arise.
     - Tools Used: **Prometheus, Grafana, ELK Stack** (Elasticsearch, Logstash, Kibana), Datadog, New Relic


#### CI/CD Pipeline - Using Jenkins & Kubernetes

```
pipeline {
    agent any
   
    environment {
        DOCKER_IMAGE = "myapp:latest"
        DOCKER_REGISTRY = "docker.io/myrepo"
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/user/repo.git'
            }
        }
        stage('Code Quality Check') {
            steps {
                sh 'sonar-scanner'
            }
        }
        stage('Build & Unit Test') {
            steps {
                sh './gradlew build test'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh "docker build -t $DOCKER_IMAGE ."
            }
        }
        stage('Push to Docker Registry') {
            steps {
                sh "docker login -u user -p password $DOCKER_REGISTRY"
                sh "docker tag $DOCKER_IMAGE $DOCKER_REGISTRY/$DOCKER_IMAGE"
                sh "docker push $DOCKER_REGISTRY/$DOCKER_IMAGE"
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh "kubectl apply -f k8s/deployment.yaml"
            }
        }
    }
    post {
        success {
            echo 'Deployment Successful!'
        }
        failure {
            echo 'Deployment Failed!'
        }
    }
}
```

 #### Terraform Azure

Terraform can provision Azure resources via main.tf

```
provider "azurerm" {
  features {}
}
resource "azurerm_resource_group" "rg" {
  name     = "my-resource-group"
  location = "East US"
}
resource "azurerm_kubernetes_cluster" "aks" {
  name                = "my-aks-cluster"
  location           = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  dns_prefix         = "myaks"
}
```

#### high-availability (HA) architecture in AWS for a critical application

- Steps
     - Use Elastic Load Balancer (ELB) to distribute traffic.
     - Deploy application instances in multiple Availability Zones (AZs) within an Auto Scaling Group for failover.
     - Store data in Amazon RDS with Multi-AZ enabled or DynamoDB Global Tables for HA.
     - Use Amazon S3 for storing static content with cross-region replication.
     - Implement Route 53 health checks for DNS failover.
     - For stateful workloads, use EFS or FSx for shared storage.

#### implement Blue-Green deployment in AWS DevOps

- Steps
     - Create two environments:
          - Blue (existing production environment)
          - Green (new version to be tested)
     - Deploy new changes to the Green environment using AWS CodeDeploy or Elastic Beanstalk.
     - Use Route 53 weighted routing or ALB Target Groups to gradually shift traffic.
     - After successful validation, switch all traffic to Green.
     - If rollback is needed, switch back to Blue instantly.

#### automate infrastructure provisioning

- types
     - Use AWS CloudFormation or Terraform to define infrastructure as code (IaC).
     - Store configurations in GitHub/CodeCommit for versioning.
     - Use AWS Systems Manager Parameter Store for managing secrets.
     - Automate deployments with AWS CodePipeline integrated with CloudFormation/Terraform.
     - Use Ansible/Puppet/Chef for configuration management. 

#### secrets management

- Steps
     - Use AWS Secrets Manager or SSM Parameter Store for securely storing secrets.
     - Ensure IAM roles have least privilege access to secrets.
     - Use AWS Lambda to rotate secrets periodically.
     - Fetch secrets dynamically using environment variables or API calls in the pipeline.

#### Monitor and log AWS DevOps

- Process
     - Use CloudWatch Metrics & Logs for monitoring applications.
     - Enable AWS X-Ray for distributed tracing.
     - Use CloudTrail for auditing API calls.
     - Store logs in Amazon S3 and analyze with AWS Athena or OpenSearch (Elasticsearch Service).
     - Use SNS or EventBridge for real-time alerting. 

#### Lambda - optimize CI/CD pipelines

- Process
     - Use AWS CodePipeline with CodeBuild and CodeDeploy.
     - Package Lambda as a ZIP file or a container image using AWS SAM/CDK.
     - Deploy Lambda with Canary or Linear Deployment in CodeDeploy.
     - Use Lambda versions & aliases for easy rollback.

#### rollback in AWS CodeDeploy

- Process
     - Use CodeDeploy Deployment Groups with automatic rollback enabled.
     - Enable CloudWatch alarms to trigger rollback if deployment fails.
     - Use Blue-Green deployment strategy for safe rollback.

#### multi-account AWS DevOps setup

- Process
     - Use AWS Organizations and Control Tower for centralized management.
     - Implement AWS IAM Identity Center (SSO) for access control.
     - Use AWS Config & Security Hub for security compliance across accounts.
     - Use AWS Transit Gateway for cross-account networking.

#### ECS service deployment failing

- Process
     - Check CloudWatch Logs for ECS task logs.
     - Use ECS Events & Service Events for failure details.
     - Validate IAM permissions for ECS tasks.
     - Verify ALB target group health status.

#### Optimize EC2 costs

- Process
     - Use Auto Scaling Groups with spot instances.
     - Implement EC2 instance scheduler for stopping non-production instances.
     - Utilize AWS Savings Plans & Reserved Instances.

#### Kubernetes deployment

- Process
     - Use EKS (Elastic Kubernetes Service).
     - Store configurations in ConfigMaps & Secrets.
     - Use Helm Charts for deployments.
     - Implement AWS ALB Ingress Controller for traffic management.

#### Patch management

- Process
     - Use AWS Systems Manager Patch Manager.
     - Schedule patching using SSM Automation documents.
     - Enable AWS Inspector for vulnerability assessments.

#### Distributed tracing in microservices

- Use AWS X-Ray for tracing requests across microservices.
- Enable X-Ray SDK in the applicatio

#### Security compliance

- Process
     - Use AWS Config Rules and AWS Security Hub
     - Implement GuardDuty for anomaly detection.
     - Scan container images with Amazon ECR Scan.

#### secure Azure DevOps pipeline

- Process
     - Use Azure DevOps Service Connections for secure authentication.
     - Store secrets in Azure Key Vault.
     - Implement pipeline approvals & gates.

#### AKS - implement CI/CD


#### Kubernetes Cluster Deployment

- We need to 
     - Automate everything using Terraform, Ansible, Helm, ArgoCD.
     - Ensure security with IAM, RBAC, Secrets Management, Network Policies.
     - Optimize observability with Prometheus, Grafana, ELK, Datadog.
     - Improve resilience with Chaos Engineering, Backup Strategies, Disaster Recovery.

1. Planning & Architecture Design
     - Assessed business requirements (e.g., high availability, security, scaling needs).
     - Chose Cloud Provider (AWS EKS, Azure AKS, or on-premises with Kubernetes).
     - Defined Cluster Topology:
          - Master & Worker Nodes placement.
          - Node Pools for different workloads (e.g., General-purpose, GPU, Spot instances).
          - Network Design (VPC, Subnets, CIDR blocks, Security Groups)
      
2. Infrastructure Provisioning (IaC)
     - Used Terraform & Ansible to automate infrastructure deployment:
          - Terraform for provisioning cloud resources (VPC, Subnets, EKS/AKS Cluster, IAM roles, Storage).
          - Ansible for configuring nodes, setting up monitoring agents, and applying security hardening.
- Example Terraform Code for AWS EKS Deployment
```
module "eks" {
  source          = "terraform-aws-modules/eks/aws"
  cluster_name    = "prod-cluster"
  cluster_version = "1.26"
  subnets         = module.vpc.private_subnets
  vpc_id          = module.vpc.vpc_id

  node_groups = {
    worker_nodes = {
      desired_capacity = 3
      max_capacity     = 5
      min_capacity     = 2
      instance_types   = ["t3.medium"]
    }
  }
}
```

3. Cluster Bootstrapping & Configuration
     - Deploying Kubernetes Control Plane & Worker Nodes:
          - Used eksctl for AWS or az aks create for Azure.
          - Set up RBAC & IAM roles for secure access control.
          - Configured Kubeconfig to connect the cluster.
          - Enabled Cluster Autoscaler & Horizontal Pod Autoscaler (HPA) for auto-scaling.
     - Example Command for EKS Cluster Creation:
          - eksctl create cluster --name prod-cluster --region us-east-1 --nodegroup-name worker-nodes --nodes 3
          - az aks create --resource-group myResourceGroup --name myAKSCluster --node-count 3 --enable-managed-identity

4. Networking & Security Setup
     - Configured Ingress Controller (NGINX/Istio) for external traffic routing.
     - Set up Network Policies to restrict pod-to-pod communication.
     - Integrated Cloud Security Best Practices:
          - IAM Roles for Service Accounts (IRSA) for pod security.
          - AWS Security Groups/NACLs for network-level protection.
          - Secrets Management with AWS Secrets Manager/HashiCorp Vault.
- Example Network Policy to Restrict Traffic
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-ingress
spec:
  podSelector:
    matchLabels:
      app: my-app
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: allowed-app
```

5. Deploying Applications & Services
     - Used Helm Charts for standardized app deployment.
     - ArgoCD/GitOps for continuous delivery.
     - Canary & Blue-Green Deployment for zero-downtime releases.
- Helm Command to Deploy an Application:
```
# Helm Command to Deploy an Application
helm install my-app ./charts/my-app
----
# ArgoCD Deployment Manifest Example
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
spec:
  source:
    repoURL: 'https://github.com/my-org/my-app.git'
    path: charts/my-app
    targetRevision: HEAD
  destination:
    server: https://kubernetes.default.svc
    namespace: my-namespace
```

6. Observability & Monitoring Setup
     - Installed Prometheus & Grafana for metrics.
     - Set up ELK/Splunk for log aggregation.
     - Configured Datadog/New Relic for APM (Application Performance Monitoring).
     - Integrated Alerting via Slack/Teams using Alertmanager.
- Example Prometheus Alert Rule for High CPU Usage:
```
groups:
- name: High CPU Usage Alert
  rules:
  - alert: HighCPUUsage
    expr: 100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: "High CPU usage on instance {{ $labels.instance }}"
```

7. Scaling, Security & Disaster Recovery
     - Configured Cluster Autoscaler & VPA (Vertical Pod Autoscaler).
     - Set up Pod Disruption Budgets (PDBs) to ensure high availability.
     - Configured AWS Backup & EFS snapshots for disaster recovery.
     - Conducted Chaos Engineering (using LitmusChaos) to test system resilience.
- Example Pod Disruption Budget Configuration:
```
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: my-app-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: my-app
```

8. Incident Management & Continuous Improvements
     - Integrated SLO/SLI/SLA tracking.
     - Set up blameless postmortems and RCAs for production issues.
     - Conducted weekly optimizations & security audits to enhance performance.
- Example SLI/SLO Definition in Prometheus:
```
- record: slo:availability
  expr: sum(rate(http_requests_total{status=~"2.."}[5m])) / sum(rate(http_requests_total[5m])) * 100
```

- Final Outcome & Impact
     - Deployment time reduced from 2 hours to 15 minutes using Terraform & CI/CD.
     - MTTR (Mean Time to Recovery) reduced by 40% with proactive monitoring.
     - Cloud cost optimized by 30% using auto-scaling & Spot instances.
     - System availability improved to 99.99% with robust DR & HA strategies.

#### Data inconsistency over grafana

- Resolution Steps
     - If Data Source Issue → Restart Prometheus, Check Scraping.
     - If Query Issue → Validate in Query Inspector, Adjust Labels.
     - If Dashboard Issue → Fix Time Range & Panel Settings.
     - If Logs Show Errors → Restart services, Check logs for failures.

- Step-by-Step Debugging for Incorrect Grafana Data
1. Verify Data Source Connectivity
     - Go to Grafana → Configuration → Data Sources
     - Check if the correct data source (Prometheus, CloudWatch, Loki, Elasticsearch, etc.) is selected.
     - Click "Test Connection"
- Fix: If the connection fails, check:
     - Network connectivity (Firewall rules, VPN, proxy issues).
     - Service availability (Is Prometheus/Elasticsearch running?).
     - Correct endpoint URL and credentials
 
2. Check if the Data Source is Collecting Fresh Data
- If using Prometheus, run a manual query to check if recent data exists:
     - curl -s http://prometheus-server:9090/api/v1/query?query=up
- If using Elasticsearch, check the last log timestamp
```
 {
  "query": { "range": { "@timestamp": { "gte": "now-5m" } } }
}
```
- Fix:
     - Restart the data source service (systemctl restart prometheus).
     - Check data retention policy (Older data may be purged).

3. Validate Grafana Query Execution
- Edit the panel → Click on Query Inspector
- Check:
     - Does the query return data?
     - Are filters or labels incorrect?
     - Time range settings (Last 5 min vs. Last 1 hour)?
- Fix:
     - Adjust filters and labels in queries.
     - Try executing the same query directly in Prometheus/Elasticsearch.
     - Increase the time range to see if data exists.

4. Check Data Freshness & Scraping Intervals
- If using Prometheus, verify scrape_interval:
```
scrape_configs:
  - job_name: 'kubernetes-nodes'
    scrape_interval: 15s  # Ensure this is not too high
    static_configs:
      - targets: ['node-exporter:9100']
```
- Fix:
     - Ensure scrape jobs are running (prometheus.yml).
     - Restart Prometheus if scrape jobs are failing.

5. Verify Grafana Dashboard Panel Settings
- Ensure "Now" option is selected instead of an old timestamp.
- In panel settings, check Y-axis units & transformation settings.
- Ensure "Relative Time" and "Time Shift" settings are not misconfigured.
- Fix:
     - Reset panel settings and reload the dashboard.
     - Change visualization type (Table, Graph, Heatmap).

6. Debug Data Source Logs
- Check Prometheus Logs for scrape errors:
     - journalctl -u prometheus --tail -n 50
- Check Grafana Logs for panel errors:
     - tail -f /var/log/grafana/grafana.log
- Fix:
     - Restart Grafana (systemctl restart grafana-server).
     - Check for rate-limiting issues in cloud-based services.

#### Pod running but Application not accessible 
- when we try to deploy one application using Kubernetes and helm, that application pod is up and running, we not seeing any errors messages but still we are unable access the application, may i know possible reason and how to resolve this issue
-  even if the pod is running, the issue can be in service, networking, application config, or DNS. We need to troubleshoot layer by layer starting from pod to ingress.

1. Service Not Created or Misconfigured
     - the first thing is to check if the Kubernetes Service is created properly.
     - Sometimes Helm charts may not create the service due to a values file issue or missing templates.
     - kubectl get svc -n <namespace>
     - f no service is there or it's misconfigured (wrong port, selector mismatch), then our request never reaches the pod.
2. Wrong TargetPort or Port Mapping
     - Even if the service is created, haaa… if the port in the service does not match the port exposed by the pod, it won’t work.
     - Check the container's exposed port in deployment and make sure it matches the targetPort in the service definition.
3. Application Not Listening on 0.0.0.0
     - This one is very tricky and easy to miss.
     - If the application inside the pod is only listening on localhost (127.0.0.1) instead of 0.0.0.0, then external traffic won’t be able to reach it even though the pod is healthy.
     - Exec into the pod and run:
          - netstat -tulnp
          - Make sure it's listening on 0.0.0.0.
4. Ingress Not Configured or DNS Issue (if using Ingress)
     - if you're using an Ingress controller, then maybe the Ingress resource is not created or the host/path is not matching.
     - Also, sometimes the DNS name we are trying to access doesn't resolve correctly.
          - kubectl get ingress -n <namespace>
          - And make sure the external IP of Ingress controller is reachable.
5. NetworkPolicy Blocking Access
     - If NetworkPolicies are applied in the cluster, they might be blocking traffic to or from the pod.
     - kubectl get networkpolicy -n <namespace>
     - If policies are present, confirm they allow traffic to the pod.
6. Firewall or Security Group Issue (If on Cloud)
     - if this is a cloud-based cluster (like EKS, GKE), then make sure the security group, firewall rules, or NACLs are allowing traffic on the required ports.
7. Liveness or Readiness Probes Misconfigured
     - Sometimes the probes are misconfigured, so Kubernetes thinks the app is ready, but actually it’s not serving traffic yet.
8. Helm Values Misconfigured
     - if we used a values.yaml file while installing the chart, we need to double-check if the port, service type, ingress settings, etc. are correct in the values file.

- Start Debugging Step-by-Step
     - kubectl get pods -o wide – Confirm pod is running and node IP.
     - kubectl get svc – Confirm service is present.
     - kubectl describe svc <svc-name> – Check port and selector match.
     - kubectl exec -it <pod> -- curl localhost:<port> – See if app responds.
     - Try kubectl port-forward and test locally:

## CRITICAL issues

#### Critical Kubernetes Outage in Production due to Misconfigured Network Policy
- During a high-traffic period, a production AKS (Azure Kubernetes Service) cluster became partially unreachable. Several microservices failed to communicate internally, causing degraded service for users. Root cause pointed to a recently applied NetworkPolicy that unintentionally blocked internal pod-to-pod communication across namespaces.
- What I Did to Fix It:
     - Immediate Action: Jumped into incident response mode. Engaged stakeholders via incident management tools (StatusPage, PagerDuty).
     - Initial Diagnosis: Used kubectl logs, kubectl describe, and networkpolicy inspection to isolate the issue to a recent change.
     - Fix: Rolled back the offending NetworkPolicy and validated internal service communication using busybox test pods and curl within the cluster.
     - Recovery: Restarted affected pods and scaled impacted services to restore stability.
- Learnings:
     - Misconfigured security resources can have large blast radius in microservices environments.
     - Lack of a NetworkPolicy dry-run and automated validation was a key gap.
     - Change was not tested in lower environments with representative traffic or topology.
- Preventive Steps Taken:
     - Implemented Change Freezes during peak business hours.
     - Integrated OPA/Gatekeeper to validate NetworkPolicy schema and logic in PR stage.
     - Created policy simulation scripts for pre-deployment validation.
     - Setup Prometheus alerts for service availability and inter-service latency to catch similar issues early.
- Automation Implemented:
     - Developed a GitHub Actions workflow that:
          - Lints and tests YAML manifests.
          - Validates NetworkPolicies against OPA policies.
          - Runs integration tests in a staging AKS cluster.
     - Built an automated Slack-based canary alert system using Prometheus and Alertmanager to catch sudden drop in pod-to-pod metrics.

#### Critical Production Deployment Failure Due to Secret Expiry in CI/CD Pipeline
- During a scheduled deployment via Jenkins to an Azure AKS cluster, the pipeline failed at the container image pull stage. Investigation revealed that the Azure Container Registry (ACR) pull secret used in the Kubernetes cluster had expired, which blocked new pods from pulling updated images.
- Immediate Response:
     - Triggered a manual redeployment using the latest image already present in a node’s cache (short-term workaround).
     - Updated the expired Kubernetes imagePullSecret with a freshly generated ACR token using Azure CLI.
     - Redeployed services with the corrected secret.
- Root Cause Analysis (RCA):
     - Found that the secret was a short-lived service principal token not configured to auto-refresh.
     - No monitoring was in place to alert on secret/token expiration.
- Learnings:
     - CI/CD and runtime environments must have proactive secret management.
     - Secrets with expiration need visibility and rotation mechanisms.
     - Failure occurred due to inadequate observability in Kubernetes authentication components.
- Preventive Steps Taken:
     - Integrated Azure Managed Identity with AKS and ACR to avoid static secrets altogether.
     - Implemented cert-manager for TLS automation and external-dns integration.
     - Configured Prometheus alerts to monitor ImagePullBackOff errors and token expiration timestamps via custom metrics.
     - Established a runbook for token validation and failover registry usage.
- Automation Implemented:
     - Developed an Ansible playbook to:
          - Validate secret expiration dates across clusters.
          - Automatically renew and patch secrets when nearing expiry.
     - Jenkins pipeline updated to include a pre-deploy check stage for verifying secret validity and image accessibility.
     - Set up Grafana dashboards for secret rotation visibility and pipeline status.

#### Critical Production Outage Due to Disk Space Exhaustion on Kubernetes Worker Node - CrashLoopBackOff
- An alert was triggered via Prometheus for high disk usage (>95%) on one AKS node. Shortly after, multiple pods began failing with CrashLoopBackOff, Evicted statuses, and kubelet logs indicated the node was under disk pressure. This caused a partial outage of a customer-facing application.
- Immediate Response:
     - Marked the node as unschedulable using kubectl cordon to stop new pods from being assigned.
     - Investigated using du, df, and journalctl — found large volumes of orphaned Docker images, logs, and tmp files.
     - Cleaned up unused Docker layers and images with docker system prune -a (in coordination with the container runtime policy).
     - Reclaimed space by compressing and archiving old logs, then restarted the kubelet to restore pod functionality.
- Root Cause:
     - The cluster didn’t have log rotation or cleanup automation.
     - Image pull policy was set to Always, with frequent deployments pulling new images, but old ones weren’t cleaned.
- Learnings:
     - Kubernetes nodes need continuous observability and maintenance.
     - Image and log bloat can creep up fast if not managed proactively.
     - Lack of proper monitoring thresholds and alert severity tuning led to delayed response.
- Preventive Steps Taken:
     - Implemented logrotate on worker nodes with daily compression and cleanup policy.
     - Created a DaemonSet job that performs regular image cleanup and tmp directory maintenance.
     - Updated kubelet eviction thresholds to prevent aggressive pod eviction behavior.
     - Introduced node affinity rules and pod anti-affinity to reduce resource contention on a single node.
- Automation Implemented:
     - Wrote a cron-based cleanup job via Kubernetes CronJob to clear temp directories and rotate logs.
     - Enhanced Prometheus node exporter metrics with custom alerts for:
          - /var/lib/docker usage
          - inode exhaustion
          - log folder growth
     - Set up Grafana dashboard panels for disk usage trends per node.
     - Configured Slack alerts for critical thresholds on disk usage before failure point (80%, 90%, etc.)

#### Critical Outage Due to Expired SSL Certificate on Production Ingress
- At midnight during routine monitoring, synthetic checks failed for all HTTPS endpoints of a production application hosted on AKS behind an NGINX Ingress Controller. Customer-facing services became unreachable due to expired TLS certificates, resulting in HTTP 526/SSL handshake failures. The certificate had expired just hours earlier.
- Immediate Response:
     - Engaged the on-call SRE team and declared a P1 incident.
     - Identified the expired certificate using kubectl describe certificate (cert-manager) and openssl s_client.
     - Manually renewed the TLS certificate from Let’s Encrypt using cert-manager and forced a reload on the Ingress pod.
     - Confirmed restoration via browser test and synthetic health checks.
- Root Cause:
     - cert-manager was configured but failed silently due to rate-limiting from Let’s Encrypt and missing email notifications.
     - Alerts for TLS expiration were not in place.
     - Certificate renewal CRD (CertificateRequest) status was stuck in Pending.
- Learnings:
     - TLS expiry should never be a surprise; there must be visibility and alerts at least a week in advance.
     - Even if cert-manager is in place, monitoring its health and CRD status is essential.
     - Human dependency in cert renewal = risk.
- Preventive Steps Taken:
     - Enabled automatic retry logic and fixed RBAC issues for cert-manager to renew certificates successfully.
     - Added Grafana dashboards for certificate expiry dates using kube-state-metrics and Prometheus rules.
     - Subscribed to Let’s Encrypt webhook notifications for renewal failures.
     - Validated all environments (staging, dev, prod) had correct cert-manager configurations with proactive renewal enabled.
- Automation Implemented:
     - Wrote a custom Prometheus alert for:
          - TLS cert expiry within 7, 3, and 1 day thresholds.
          - Failed CertificateRequest or stuck Order statuses from cert-manager.
     - Deployed a daily Kubernetes CronJob that runs openssl checks on ingress endpoints and posts results to a Slack channel.
     - Used Ansible to validate and rotate TLS secrets in all environments.

#### Critical High Latency and Timeouts Due to AKS Cluster Auto-Scaling Failure During Traffic Spike
- During a flash sale on our e-commerce platform, users began experiencing long response times and timeouts on checkout and product detail pages. Prometheus alerts fired for high CPU and memory usage across multiple pods. However, Horizontal Pod Autoscaler (HPA) and Cluster Autoscaler did not scale the application as expected.
- Immediate Response:
     - checked CPU and memory usage via Grafana dashboards and Prometheus queries.
     - Verified HPA status using kubectl get hpa — found that the pod replica count was stuck despite metrics exceeding target thresholds.
     - Identified that Cluster Autoscaler could not provision new nodes due to Azure resource quota limits being hit in the region.
     - Temporarily mitigated by manually increasing pod replicas and restarting underperforming pods to rebalance load.
- Root Cause:
     - HPA was correctly configured but couldn’t scale because Cluster Autoscaler failed to add nodes.
     - Azure VM quota for the node SKU was exhausted, and no alerting or fallback mechanism was in place.
     - There was also a delay in metrics being reported to HPA due to misconfigured metrics server throttling under load.
-  Learnings:
     - Auto-scaling is only effective if all the dependent systems (quotas, metrics, scale limits) are aligned.
     - Relying on default quota values in cloud environments is risky for production workloads.
     - Monitoring autoscaler logs and metrics behavior under real load is crucial
- Preventive Steps Taken:
     - Worked with Azure support to increase VM quota proactively in all production regions.
     - Implemented a capacity planning policy for peak events with scheduled pre-scaling based on business events.
     - Configured KEDA (Kubernetes Event-driven Autoscaling) for more responsive event-based scaling on specific workloads like queue consumers.
     - Tuned metrics server resource limits and HPA polling intervals for better responsiveness.
- Automation Implemented:
     - Wrote Terraform scripts to periodically validate Azure quota levels and notify via Slack if thresholds are near.
     - Enhanced Prometheus alerts to include:
          - HPA stuck in scaling condition
          - Cluster Autoscaler failure events (via logs or node count stagnation)
     - Added a pre-event scaling job triggered via Jenkins or Azure DevOps before planned campaigns.
     - Created Grafana heatmaps to visualize scaling events versus application traffic in real time.

#### Critical Git Rebase Gone Wrong — Lost Commits in Shared Branch With No Backup
- A developer attempted to clean up commit history using git rebase -i on a shared feature branch used by multiple team members. Unfortunately, during the interactive rebase, several commits containing production-related Terraform and Helm chart changes were accidentally squashed and dropped. The rebase was force-pushed (git push -f) without creating a backup or new branch. Several commits were overwritten, resulting in missing infrastructure changes and breaking the downstream CI/CD pipeline.
- Immediate Response:
     - Notified all team members via Slack to stop work on the affected branch.
     - Cloned the branch from another developer’s local repo who had not yet pulled the latest (broken) version — effectively recovering the lost commits from reflog.
     - Used git reflog and git fsck to trace and recover orphaned commits from the rebased history.
     - Restored the original state of the branch by pushing a recovered version to a new branch (feature/recovery-branch) and rebased the fixed history back into develop.
- Root Cause:
     - Git rebase was performed without understanding the implications of squash and drop during rebase -i.
     - There was no backup branch created before rewriting history.
     - No peer-review or pre-check before force pushing to a shared branch — breaking the golden rule of Git collaboration.
- Learnings:
     - Rebase is powerful but dangerous on shared branches — should never be done without coordination or backups.
     - Git reflog can save the day — but only within a short time window and if local history is intact.
     - Rewriting history on shared branches breaks collaboration and CI/CD workflows.
- Preventive Steps Taken:
     - Enforced a policy: No git push -f allowed on shared branches via GitLab/GitHub branch protection rules.
     - Mandated creation of a backup branch before any rebase or history rewrite.
     - Introduced a pre-rebase checklist including branch backup, peer review, and isolation of experimental rebases to personal branches.
     - Documented a Git recovery playbook with steps to recover from lost commits using reflog, cherry-pick, or git fsck.
- Automation Implemented:
     - Wrote a Git pre-push hook that warns users when they’re about to push forcefully to a protected/shared branch.
     - Added GitLab/GitHub CI pipeline check to block force pushes on main, develop, and other shared branches.
     - Used Terraform Cloud remote state locking and tagging to avoid similar infra state drift caused by lost commits.
     - Enabled daily backup of Git branches to a mirrored repo using a scheduled GitHub Actions job.

-----------------------------

##### IAC and it's benefits
- I have hands-on experience with Infrastructure as Code (IaC) primarily using Terraform and ARM templates in Azure environments. IaC has been a critical part of my workflow in managing infrastructure in a reliable, repeatable, and automated manner.
- For example, I used Terraform to provision and manage Azure resources such as virtual networks, load balancers, virtual machines, and storage accounts. I modularized the Terraform codebase to ensure reusability and maintainability across multiple environments like dev, staging, and production.
- We implemented version control for our IaC code in Git, enabling proper change tracking and peer reviews through pull requests. Combined with a CI/CD pipeline in Azure DevOps, any infrastructure changes went through automated validation and testing before being applied.

- Benefits of IaC:
1. Consistency and Standardization:
     - Since infrastructure is defined in code, every environment (dev, QA, prod) can be provisioned identically, reducing configuration drift.
2. Automation and Speed:
     - Provisioning infrastructure becomes fast and repeatable, eliminating manual steps and reducing time-to-deploy.
3. Version Control and Auditability:
     - Infrastructure definitions stored in Git can be reviewed, audited, and rolled back if needed.
4. Improved Collaboration:
     - Teams can work together on infrastructure the same way they work on application code, using the same tools and processes (e.g., Git workflows, code reviews).
5. Disaster Recovery and Scalability:
     - With IaC, rebuilding an entire environment is fast and easy, which helps with disaster recovery scenarios or scaling up on demand.

- when we needed to onboard a new application environment, a developer would raise a PR with the required changes — such as creating new Azure Resource Groups, Key Vaults, or App Services. The PR triggered a CI pipeline in Azure DevOps that did the following:
1. Terraform Format & Validate:
     - Ensured the code was syntactically correct and followed formatting standards.
2. Terraform Plan:
     - Ran a dry-run to show what changes would be made in the infrastructure. This plan was output as an artifact and attached to the PR for review.
3. Static Code Analysis:
     - We integrated tools like tflint and checkov to detect misconfigurations or security risks early.
- Once approved and merged, the CD pipeline kicked in:
1. Terraform Apply (with approval):
     - Applied the infrastructure changes in a controlled non-prod environment. For production changes, we required manual approval via an Azure DevOps environment gate.
2. Post-deployment Validation:
     - Simple checks like resource existence, tagging compliance, and network connectivity were automated as part of post-deploy jobs.

- Example scenario:
     - We had a case where a developer mistakenly tried to delete a shared resource through a misconfigured module. Thanks to the Terraform Plan step, this was caught before merging. The reviewer flagged it, the developer corrected the code, and we avoided a potential production outage — all due to proper version control and validation in the pipeline.

#### IaC Workflow with Git + Azure DevOps Pipelines

```
     Developer
         │
         ▼
  ┌──────────────┐
  │  Git Commit  │
  └──────┬───────┘
         │
         ▼
  ┌────────────────────┐
  │ Pull Request (PR)  │
  └────────┬───────────┘
           │
           ▼
  ┌────────────────────────────────────────┐
  │ CI Pipeline (Triggered on PR)         │
  │  - terraform fmt                      │
  │  - terraform validate                 │
  │  - tflint / checkov (static checks)   │
  │  - terraform plan (save as artifact)  │
  └────────────────────────────────────────┘
           │
           ▼
   PR Review & Approval (Plan Reviewed)
           │
           ▼
     Merge to main branch
           │
           ▼
  ┌────────────────────────────────────────┐
  │ CD Pipeline (Triggered on merge)       │
  │  - terraform init                      │
  │  - terraform apply (non-prod)          │
  │  - approval gate for prod              │
  │  - post-deployment checks              │
  └────────────────────────────────────────┘
```


#### optimize cicd pipeline
- I focus on:
     - Parallelization: Run unit tests and builds in parallel.
     - Caching: Use pipeline/task-level caching (e.g., Docker layers, dependency caching).
     - Selective builds: Use path filters to trigger builds only on relevant changes.
     - Artifact reuse: Avoid rebuilding unchanged artifacts.
     - Security scanning and linting: Integrated early in the pipeline.
     - Monitoring and alerting: To catch slow steps or failures proactively.

#### Achieve containerization
- I use Docker to containerize applications by writing efficient Dockerfiles and structuring multi-stage builds for lean images. I ensure:
     - Minimal base images (e.g., alpine)
     - Proper .dockerignore usage
     - Externalized configurations via environment variables
     - Store and version images in Azure Container Registry (ACR)

#### manage secrets
- Azure Key Vault: Integrated with pipelines to inject secrets securely.
- Environment variables: Passed securely at runtime without hardcoding.
- Git secrets scanning: Prevent accidental commits of sensitive info.
- RBAC: Limit access based on least privilege.

#### monitoring and logging in distributed systems
- Monitoring: Use Azure Monitor, Prometheus, and Grafana to track CPU, memory, network, and custom metrics.
- Logging: Use ELK stack, Fluentd, or Azure Log Analytics for centralized log collection.
- Tracing: Implement distributed tracing using OpenTelemetry.
- Alerting: Set up alert rules for anomalies, thresholds, and failures.

#### rollback and disaster recovery
- Blue-Green and Canary deployments: Enable safe rollback to previous version.
- Versioned artifacts: Maintain old versions for quick rollback.
- Backups: Regular DB and stateful app backups.
- Disaster recovery plan: Runbook, replication (e.g., Azure Site Recovery), and failover testing.

#### Integrate DevSecOps practices
- Shift Left Security: Incorporate static code analysis (e.g., SonarQube), SAST/DAST early.
- Dependency scanning: Use tools like OWASP Dependency-Check or Snyk.
- Container scanning: Scan Docker images (e.g., Trivy, Azure Defender).
- RBAC & Policy as Code: Enforce with OPA or Azure Policy.

#### Cloud-native technologies, manage DevSecOps environment
- Experience with Kubernetes (AKS), service mesh (Istio), Helm, and serverless (Azure Functions). In DevSecOps:
     - Manage with GitOps (ArgoCD)
     - Automate deployments with Helm
     - Monitor via Prometheus/Grafana
     - Secure with admission controllers, network policies, and Pod Security Standards.

#### Shift Left
- Shift Left" means integrating security, testing, and compliance earlier in the software development lifecycle. Importance:
     - Detects issues sooner, reducing cost and time to fix.
     - Builds security mindset into the development process.
     - Enhances CI/CD by catching vulnerabilities before deployment.

#### zero-downtime deployment strategy
- Zero-Downtime:
     - Blue-Green deployments
     - Canary releases
     - Feature toggles
     - Readiness/liveness probes

#### designing and implementing a scalable and resilient infrastructure
- Scalable and Resilient Infrastructure:
     - Auto-scaling for VMs or pods
     - Load balancers and circuit breakers
     - Multi-region deployments
     - Use infrastructure as code and monitoring/alerting for quick incident response.

#### GitOps 
- GitOps uses Git as the single source of truth for both application code and infrastructure.
- GitOps vs Traditional DevOps:
     - GitOps: Pull-based, declarative, automated sync (e.g., ArgoCD/Flux).
     - DevOps: Often push-based, imperative or script-driven.
     - GitOps offers better auditability, rollback, and compliance.
     - Azure services; Azure Virtual Machine Scale Sets, and Azure management and governance features and tools

#### High Latency in Production API
- Issue: A customer-facing API experiences a sudden increase in latency, leading to poor user experience. The issue occurs intermittently, but it’s becoming more frequent.
- **Investigation:**
     - Monitor logs and metrics: Used Prometheus + Grafana to check API response times.
     - Load analysis: Found that requests spiked before latency increased.
     - Database queries: Observed slow queries in PostgreSQL logs.
     - Infrastructure: Checked Kubernetes pod resource usage and noticed high CPU throttling.
- **Resolution:**
     - Optimized slow queries by adding indexes and reducing joins.
     - Increased CPU limits for Kubernetes pods to prevent throttling.
     - Enabled autoscaling to handle load spikes dynamically.
     - Implemented circuit breakers using Istio to handle failures gracefully.


#### Outage Due to Expired TLS Certificate
- Issue: A service goes down suddenly, returning TLS handshake failure errors. Customers report that they can’t connect.
- **Investigation:**
     - Checked logs: Found certificate expired error in Nginx logs.
     - Verified certificate: Ran openssl s_client to check expiry.
     - Looked at renewal process: Found that the Let’s Encrypt auto-renewal cron job had failed silently.
- **Resolution:**
     - Manually renewed the certificate to restore service (certbot renew).
     - Fixed cron job issues and added alerting if renewal fails.
     - Enabled certificate monitoring to avoid future outages using Cert-Manager in Kubernetes.

#### Kubernetes Pod CrashLoopBackOff
- Issue: A microservice running in Kubernetes is continuously restarting with CrashLoopBackOff status.
- **Investigation:**
     - Ran kubectl logs <pod> and found OOMKilled (Out of Memory).
     - Checked kubectl describe pod <pod> and saw memory limit exceeded.
     - Verified resource requests/limits in the Helm chart.
- **Resolution:**
     - Increased memory limits for the pod in Kubernetes YAML.
     - Implemented horizontal pod autoscaling (HPA) to distribute load.
     - Optimized the application to use less memory (e.g., reducing in-memory cache size).
 
#### Database Deadlocks Causing Downtime
- Issue: A high-traffic e-commerce platform experiences frequent database deadlocks, causing API timeouts.
- Investigation:
     - Checked database slow query logs and found frequent deadlocks.
     - Analyzed query patterns and noticed multiple updates to the same rows.
     - Reviewed transaction isolation levels and found unnecessary use of SERIALIZABLE.
- **Resolution:**
     - Changed transactions to use lower isolation levels (READ COMMITTED).
     - Refactored queries to reduce lock contention by using optimistic locking.
     - Introduced caching (Redis) to reduce database load.

#### CICD Pipeline Failure Before Deployment
- Issue: A CI/CD pipeline (GitHub Actions + ArgoCD) fails before deploying to production.
- Investigation:
     - Checked CI logs: Found build failures due to a missing dependency.
     - Reviewed Dockerfile: Saw an outdated base image that no longer exists.
     - Checked Kubernetes manifest changes: Found a misconfigured ConfigMap reference.
- **Resolution:**
     - Updated Docker base image to a maintained version.
     - Fixed the ConfigMap reference in kustomization.yaml.
     - Added unit tests & integration tests to prevent future failures.

#### DDoS Attack Causing Service Downtime
- Issue: A critical service is suddenly unreachable, with a spike in traffic from unknown sources. The CPU and memory of the servers are maxed out.
- **Investigation:**
     - Checked cloud provider logs (AWS WAF, GCP Cloud Armor, or Cloudflare).
     - Noticed a huge number of requests from specific IP ranges.
     - Found suspicious traffic patterns (e.g., thousands of requests per second to a single API endpoint).
- **Resolution:**
     - Blocked malicious IPs using a Web Application Firewall (WAF).
     - Rate-limited requests with Nginx and API Gateway.
     - Enabled autoscaling to absorb sudden load spikes.
     - Implemented bot detection to block automated traffic.

#### Kafka Consumers Lagging Behind
- Issue: A Kafka-based event-driven service is processing messages too slowly, causing lag to accumulate.
- **Investigation:**
     - Used kafka-consumer-groups.sh to check consumer lag.
     - Found high backlog in a few partitions.
     - Monitored consumer logs and saw timeouts and slow processing.
- **Resolution:**
     - Increased the number of consumer instances to parallelize processing.
     - Tuned Kafka fetch size and batch sizes to improve efficiency.
     - Optimized the application to process messages faster (e.g., by reducing external API calls).
     - Verified offset commits to ensure consumers weren’t getting stuck.

#### Incident Response for Database Disk Full
- Issue: A PostgreSQL database stops working because the disk is 100% full.
- **Investigation:**
     - Checked df -h and found the primary disk was out of space.
     - Ran du -sh * to identify large files and found huge WAL logs.
     - Looked at pg_stat_activity and saw long-running queries not vacuuming properly.
- **Resolution:**
     - Archived WAL logs to S3 and cleaned up old files.
     - Expanded disk size on the cloud provider (AWS/GCP/Azure).
     - Tuned autovacuum settings to prevent future bloat.
     - Implemented disk usage monitoring alerts using Prometheus

#### Kubernetes Nodes Marked as NotReady
- Issue: Multiple Kubernetes nodes go into NotReady state, causing service disruption.
- **Investigation:**
     - Ran kubectl get nodes and saw multiple nodes in NotReady.
     - Checked kubectl describe node <node> and found disk pressure and kubelet errors.
     - Logged into the node and found /var/lib/kubelet filled with orphaned containers.
- **Resolution:**
     - Cleared orphaned Docker containers using docker system prune.
     - Resized the node's disk storage in cloud settings.
     - Configured Node Autoscaler to prevent future resource exhaustion.
     - Set up alerts for disk usage and container failures.

#### Deployment Rollback After a Failed Release
- Issue: A new deployment to Kubernetes introduces a bug, causing 500 errors in production.
- **Investigation:**
     - Checked logs using kubectl logs -f <pod>.
     - Found a new environment variable missing, causing an application failure.
     - Validated with kubectl get configmap and saw an incorrect value.
- **Resolution:**
     - Used kubectl rollout undo deployment <deployment> to roll back to the previous working version.
     - Fixed the ConfigMap issue and redeployed safely.
     - Added pre-deployment checks in CI/CD to catch missing configurations.

#### Memory Leak in a Microservice
- Issue: A Go-based microservice running in Kubernetes keeps crashing due to out-of-memory (OOM) errors.
- **Investigation:**
     - Used kubectl top pods to check memory consumption.
     - Found a single pod consuming 5x more memory than expected.
     - Ran pprof to analyze memory usage and discovered unreleased goroutines causing memory leaks.
- **Resolution:**
     - Fixed the code to properly close goroutines and release memory.
     - Introduced heap profiling with pprof for future debugging.
     - Set memory limits in Kubernetes to prevent crashes.

#### CICD Pipeline Slows Down Due to Large Docker Images
- Issue: Developers complain that CI/CD pipelines take too long to deploy due to large Docker images.
- **Investigation:**
     - Analyzed Docker layers with docker history.
     - Found that the base image included unnecessary libraries.
     - Saw multiple COPY commands that increased image size.
- **Resolution:**
     - Switched to a lighter base image (e.g., Alpine instead of Ubuntu).
     - Used multi-stage builds to keep final images smaller.
     - Cached dependencies properly to speed up builds.

#### AWS S3 Latency Issues Impacting Web Application
- Issue: Users report slow loading times for images and assets stored in S3.
- **Investigation:**
     - Used AWS CloudWatch to check S3 request latencies.
     - Found that S3 requests were being routed to a different region.
     - Identified a misconfigured S3 bucket policy causing unnecessary redirects.
- **Resolution:**
     - Moved assets to an S3 bucket in the correct region.
     - Implemented CloudFront CDN caching to reduce latency.
     - Set lifecycle rules to optimize storage performance.     

#### Redis Connection Issues Leading to API Failures
- Issue: A Node.js application using Redis experiences intermittent connection timeouts.
- **Investigation:**
     - Checked Redis logs and saw maxclients errors.
     - Used redis-cli info to check connection stats.
     - Found that application instances weren’t closing idle Redis connections.
- **Resolution:**
     - Increased maxclients in Redis to allow more concurrent connections.
     - Added connection pooling with ioredis to prevent excessive connections.
     - Set timeouts and retries to handle transient Redis failures.

#### Slow Elasticsearch Queries Affecting Search Performance
- Issue: A search feature using Elasticsearch becomes slow, taking 5+ seconds to return results.
- **Investigation:**
     - Used /_cat/indices?v to check index health.
     - Found high shard count, leading to excessive overhead.
     - Checked /_cat/nodes?v and noticed high CPU usage on Elasticsearch master nodes.
- **Resolution:**
     - Rebalanced shards to optimize distribution.
     - Reduced replica count on low-traffic indices.
     - Used Elasticsearch caching for frequently queried data.
     - Added query profiling to optimize slow searches.

#### Cloudflare 502 Bad Gateway Errors on Production
- Issue: Users report frequent 502 errors when accessing the website through Cloudflare.
- **Investigation:**
     - Checked Cloudflare analytics and found increased 502 errors.
     - Looked at origin server logs and saw timeouts on API requests.
     - Used curl -v -H "Host: example.com" <origin-IP> to test direct access.
     - Found that the backend Kubernetes ingress controller was overloaded.
**Resolution:**
     - Increased backend timeouts in Nginx ingress.
     - Enabled origin keep-alive to reuse connections.
     - Deployed Cloudflare Argo Smart Routing for optimized traffic.
     - Scaled out backend API pods to handle increased load.

#### AWS RDS Failover Causes Brief Downtime
- Issue: A production PostgreSQL RDS instance in AWS fails over unexpectedly, causing brief downtime.
- **Investigation:**
     - Checked AWS CloudWatch and found DB instance rebooted due to high CPU.
     - Queried pg_stat_activity and found long-running queries blocking writes.
     - Verified the Multi-AZ failover event in AWS.
- **Resolution:**
     - Optimized slow queries and added indexing to improve performance.
     - Implemented connection pooling (PgBouncer) to handle high loads.
     - Enabled read replicas to reduce the load on the primary instance.
     - Configured automatic retries in the application to handle failovers.

#### Kubernetes NetworkPolicy Blocks Internal Communication
- Issue: A newly deployed service cannot communicate with other services in the cluster.
- **Investigation:**
     - Used kubectl logs <pod> and found connection refused errors.
     - Ran kubectl describe networkpolicy <policy-name> and saw a restrictive deny-all policy.
     - Used kubectl get pods -o wide to check if the pod was scheduled correctly.
- **Resolution:**
     - Updated NetworkPolicy to allow egress and ingress for the affected service.
     - Verified connectivity using kubectl exec <pod> -- curl <service-url>.
     - Implemented Pod-to-Pod NetworkPolicy rules for better security.

#### Out-of-Sync Replica in MongoDB Cluster
- Issue: A MongoDB replica set member falls behind in replication, causing stale reads.
- **Investigation:**
     - Ran rs.status() and saw one secondary was lagging by 10 minutes.
     - Checked mongod.log and found write locks were slowing replication.
     - Noticed high disk I/O on the affected node.
- **Resolution:**
     - Increased write concern for critical operations to reduce replication lag.
     - Moved the affected replica to an SSD-backed instance.
     - Tuned journal settings to optimize writes.
     - Enabled oplog size monitoring to prevent replication issues.

#### Azure AKS Cluster Autoscaler Not Working
- Issue: Kubernetes pods remain in Pending state despite autoscaling being enabled.
- **Investigation:**
     - Ran kubectl get events and found "Insufficient CPU" errors.
     - Used kubectl get nodes and saw no new nodes were being added.
     - Checked AKS logs and found a quota limit was hit.
- **Resolution:**
     - Increased VM quota limits in Azure.
     - Adjusted cluster autoscaler settings for faster scale-up.
     - Optimized resource requests to fit existing nodes better.

#### Application Logs Not Appearing in ELK (Elasticsearch, Logstash, Kibana)
- Issue: Developers report that application logs are not showing up in Kibana.
- **Investigation:**
     - Checked Logstash logs and found backpressure issues.
     - Used curl -XGET 'localhost:9200/_cluster/health?pretty' and saw Elasticsearch red status.
     - Found that log ingestion was overwhelming Elasticsearch disk space.
- **Resolution:**
     - Increased disk space and added more Elasticsearch data nodes.
     - Set up log retention policies to delete old logs.
     - Implemented log sampling to reduce excessive log volume

#### CICD Deployment Stuck Due to Kubernetes Helm Lock
- Issue: A Helm-based deployment hangs, and developers can’t release updates.
- **Investigation:**
     - Ran helm list and saw a pending release.
     - Used kubectl get pods and found init containers were stuck.
     - Checked kubectl describe pod and saw a failed ConfigMap mount.
- **Resolution:**
     - Deleted the stuck Helm release using helm rollback <release-name>.
     - Fixed the broken ConfigMap reference.
     - Enabled Helm automatic rollback for failed deployments.

#### RabbitMQ Queue Backlog Causes API Slowness
- Issue: A RabbitMQ queue accumulates millions of messages, slowing down API responses.
- **Investigation:**
     - Ran rabbitmqctl list_queues and found one queue was massively overloaded.
     - Used rabbitmqctl list_consumers and saw some consumers were down.
     - Checked dead-letter queues (DLQs) and found many messages were failing.
- **Resolution:**
     - Restarted failed consumers to start processing messages again.
     - Optimized message batching to process events more efficiently.
     - Implemented dead-letter queue reprocessing to prevent message buildup.

#### Terraform Apply Fails Due to Drift
- Issue: A Terraform apply command fails because the state file is out of sync with the infrastructure.
- **Investigation:**
     - Ran terraform plan and saw unmanaged changes.
     - Used terraform state list and found missing resources.
     - Discovered that a manual change was made in AWS, causing drift.
- **Resolution:**
     - Ran terraform import to sync the manually created resources.
     - Implemented GitOps for Terraform to prevent out-of-band changes.
     - Set up Terraform Cloud state locking to avoid conflicts.

#### Nginx Rate Limiting Blocks Legitimate Traffic
- Issue: A recently implemented Nginx rate limit blocks real users, causing HTTP 429 errors.
- **Investigation:**
     - Checked nginx.conf and found aggressive limit_req settings.
     - Reviewed logs and saw legitimate users hitting the limit.
     - Noticed API calls with high concurrency were getting blocked.
- **Resolution:**
     - Increased rate limits to match normal user behavior.
     - Implemented user-specific quotas instead of global rate limiting.
     - Used Redis-based rate limiting for more flexibility.

#### AWS Lambda Cold Start Delays
- Issue: A serverless Lambda function experiences high latency due to cold starts.
- **Investigation:**
     - Checked AWS X-Ray traces and saw long initialization times.
     - Found the function was using a large container image.
     - Discovered the function wasn’t warm before traffic spikes.
- **Resolution:**
     - Reduced the Lambda package size by switching to a smaller runtime.
     - Used Provisioned Concurrency to keep functions warm.
     - Optimized dependencies to load faster.

#### Kubernetes Horizontal Pod Autoscaler (HPA) Not Scaling
- Issue: A service under high load isn’t scaling, even though HPA is enabled.
- **Investigation:**
     - Used kubectl describe hpa and saw that CPU usage wasn’t crossing the threshold.
     - Checked kubectl top pods and noticed CPU metrics weren’t being collected.
     - Found that metrics-server wasn’t running properly.
- **Resolution:**
     - Restarted metrics-server to restore monitoring.
     - Adjusted HPA threshold to react more dynamically.
     - Enabled custom metrics scaling for better accuracy

####  fault-tolerant architecture for a high-traffic application
- To ensure high availability (HA) and fault tolerance:
     - Microservices-based architecture with Kubernetes (EKS, AKS, GKE).
     - Load Balancers (AWS ALB/ELB, Nginx, HAProxy) for distributing traffic.
     - Auto-scaling (Kubernetes HPA/VPA, AWS Auto Scaling).
     - Multi-region deployment with cloud providers for failover.
     - Database replication & sharding (RDS Multi-AZ, MongoDB Replicas).

#### key components of a DevOps architecture
- DevOps architecture includes:
     - Version Control: Git, GitHub, GitLab, Bitbucket
     - CI/CD Pipeline: Jenkins, GitHub Actions, GitLab CI/CD, ArgoCD
     - Infrastructure as Code (IaC): Terraform, Ansible, CloudFormation
     - Containerization & Orchestration: Docker, Kubernetes, Helm
     - Monitoring & Logging: Prometheus, Grafana, ELK, Datadog
     - Cloud Providers: AWS, Azure, Google Cloud (GCP)
     - Security & Compliance: DevSecOps, HashiCorp Vault, SAST/DAST tools

#### implement a zero-downtime deployment
- steps
     - Blue-Green Deployment: Two environments (blue=active, green=staging) & switch traffic via load balancer.
     - Canary Deployment: Deploy to a small % of users before full rollout.
     - Rolling Updates: Gradually replace old pods with new ones in Kubernetes.
     - Feature Flags: Enable/disable features dynamically without redeploying.

#### Secrets in a DevOps pipeline
- points
     - Vault-Based Solutions: HashiCorp Vault, AWS Secrets Manager, Azure Key Vault.
     - Kubernetes Secrets: Encrypted at rest with RBAC.
     - Environment Variables: Store secrets outside the repo.
     - GitHub/GitLab Secrets: Secure storage for CI/CD pipelines.

#### handle Kubernetes pod failures
- steps
     - Use Probes: livenessProbe & readinessProbe.
     - Auto-scaling: Kubernetes HPA scales pods based on CPU/memory.
     - Cluster Autoscaler: Adds/removes nodes based on demand.
     - Restart Policy: Always, OnFailure, Never in pod specs.

#### implement observability in DevOps
- Points
     - Metrics: Prometheus for system metrics (CPU, memory).
     - Logging: ELK (Elasticsearch, Logstash, Kibana), Loki for logs.
     - Tracing: OpenTelemetry, Jaeger for distributed tracing.
     - Alerting: PagerDuty, Prometheus AlertManager, Datadog

#### secure a DevOps pipeline
- Points:
     - Code Scanning: SAST (SonarQube), DAST (OWASP ZAP).
     - IAM Best Practices: Least privilege for AWS/GCP users.
     - Container Security: Use Trivy, Grype to scan images.
     - RBAC in Kubernetes: Restrict access to sensitive resources.

#### disaster recovery in cloud architecture
- Points :
     - Automated Backups: RDS Snapshots, EBS Snapshots.
     - Multi-Region Deployment: Active-Active, Active-Passive setups.
     - Chaos Engineering: Simulate failures using Chaos Monkey, LitmusChaos.
     - Failover Mechanisms: Route 53 DNS failover, Load balancer health checks.

#### cost optimization in DevOps
- points
     - Auto-scaling: Reduce idle resources.
     - Spot Instances: Use AWS EC2 Spot for non-critical workloads.
     - Resource Quotas: Prevent over-provisioning in Kubernetes.
     - Monitor Usage: AWS Cost Explorer, Azure Cost Management.

#### design a multi-cloud architecture for high availability?
- A multi-cloud architecture ensures fault tolerance and reduces vendor lock-in. Best practices:
     - Global Load Balancing: Use Cloudflare, AWS Route 53, or GCP Traffic Director.
     - Multi-Region Deployments: Deploy Kubernetes clusters in AWS (EKS), Azure (AKS), and GCP (GKE).
     - Data Replication: Use Google Spanner, AWS Aurora Global, or CockroachDB for multi-region DB replication.
     - CI/CD Pipeline: Use GitOps (ArgoCD, FluxCD) for consistency across clouds.

#### microservices communicate efficiently in a distributed architecture
- To ensure low latency and reliable communication between microservices:
     - Service Mesh: Use Istio or Linkerd for traffic control, retries, and observability.
     - Event-Driven Architecture: Use Kafka, RabbitMQ, or AWS SNS/SQS for async messaging.
     - API Gateway: Deploy Kong, Apigee, or AWS API Gateway to centralize API requests.
     - gRPC over REST: Use gRPC for fast, binary communication when services are internal.

#### implement a disaster recovery (DR) strategy for a mission-critical application?
- A disaster recovery plan should cover:
     - Backup & Restore: Use AWS Backup, Velero for Kubernetes, and RDS snapshots.
     - Active-Passive Architecture: Run services in primary & secondary regions, failover via Route 53 DNS.
     - Chaos Engineering: Test resilience using Gremlin, Chaos Monkey, or LitmusChaos.
     - RTO & RPO Definition: Define Recovery Time Objective (RTO) and Recovery Point Objective (RPO).

#### implement GitOps in a Kubernetes environment
- GitOps automates deployments using Git as the single source of truth.
     - Use ArgoCD or FluxCD: Automatically sync Kubernetes manifests.
     - Separate Repositories: Maintain app code & infrastructure code separately.
     - Declarative IaC: Store Terraform/Kubernetes manifests in Git.
     - Automated Rollbacks: ArgoCD can roll back failed deployments.

#### optimize CI/CD pipelines for large-scale applications
- Steps :
     - Parallel Execution: Run tests & builds in parallel (Jenkins, GitHub Actions).
     - Build Caching: Use Docker layer caching & dependency caching (npm, pip, Maven).
     - Incremental Builds: Only build changed modules (e.g., monorepo setups).
     - Self-Hosted Runners: Use GitHub Actions Self-Hosted Runners for faster performance.

#### implement a secure Infrastructure as Code (IaC) pipeline?
- Steps
     - Linting & Security Scanning: Use Checkov, tfsec for Terraform, and Ansible Lint.
     - Plan & Approval Stages: Use terraform plan before applying changes.
     - State Management: Store Terraform state in AWS S3 with DynamoDB locking.
     - Least Privilege IAM: Use service accounts with minimal permissions.

#### manage multi-environment deployments (dev/staging/prod) in Terraform
- Steps : 
     - Use Terraform Workspaces: Separate states for each environment.
     - Directory Structure: Use terraform/dev, terraform/staging, terraform/prod.
     - Backend State Separation: Store Terraform state separately per environment.
     - Use Variables: Manage environment-specific configurations with terraform.tfvars.

#### design a Kubernetes cluster for scalability and reliability
- Steps : 
     - Auto-scaling: Use HPA (Horizontal Pod Autoscaler) & Cluster Autoscaler.
     - Resource Limits: Set CPU/Memory requests & limits to prevent noisy neighbors.
     - Pod Disruption Budgets (PDBs): Ensure minimum service availability.
     - Node Affinity & Taints/Tolerations: Distribute workloads efficiently.
     - Multi-Cluster Management: Use Rancher, KubeFed, or Istio Multi-Cluster.

#### Kubernetes networking issues
- Steps :
     - Check Service & Pod Connectivity: kubectl exec -it <pod> -- curl <service>
     - Use kubectl get events: Identify networking failures.
     - Check Network Policies: Ensure correct ingress/egress rules.
     - Verify CNI Plugin Health: (calicoctl node status, weave --status)
     - Use traceroute & tcpdump: Debug network paths & dropped packets.

#### monitoring and observability in DevOps
- info
     - Metrics: Prometheus, Grafana, AWS CloudWatch, Datadog.
     - Logging: ELK Stack (Elasticsearch, Logstash, Kibana), Loki.
     - Tracing: OpenTelemetry, Jaeger for distributed tracing.
     - Alerting: PagerDuty, AlertManager, OpsGenie.

#### secure a Kubernetes cluster
- Points :
     - RBAC (Role-Based Access Control): Assign least privilege roles.
     - Pod Security Policies: Restrict root access & enforce network policies.
     - Use Secrets Properly: Store secrets in Kubernetes Secrets or HashiCorp Vault.
     - Network Segmentation: Enforce NetworkPolicies to isolate services.
     - Scan Container Images: Use Trivy, Clair, or Anchore.

#### reduce Kubernetes startup time for large applications
- Points
     - Optimize Readiness & Liveness Probes: Ensure pods don’t restart unnecessarily.
     - Use Pre-Warmed Containers: Reduce cold start times.
     - Efficient Logging: Use structured logs instead of excessive stdout logging.
     - Optimize Resource Requests: Prevent over-provisioning.

#### security-first DevOps strategy (DevSecOps)
- points
     - Shift Left Security: Perform code scans during development (SAST).
     - Runtime Security: Use Falco, Aqua Security for container security.
     - IAM & Access Control: Enforce least privilege across AWS, GCP, Azure.
     - Secrets Management: Store secrets securely with HashiCorp Vault.

#### SLO, SLA, SLI & Error Budget
1. **SLA (Service Level Agreement)** - its a business-level commitments based on SLOs, often with penalties. **External commitment (contract between provider & customer).**
   - Example: "99.95% uptime per month, or a 5% refund."
   - Use: Ensures accountability; failing an SLA can have financial/legal consequences.
2. **SLO (Service Level Objective)** - A target value for an SLI that defines acceptable performance.  **Internal goal (target reliability, e.g., 99.99% uptime).**
   - Example: "99.9% uptime over 30 days" or "p99 latency < 500ms".
   - Use: Sets a goal for reliability and user experience.
3. **SLI (Service Level Indicator)** - A quantitative measure of a system's performance. - the raw metrics - **Measured metric (actual 99.97% uptime recorded).**
   - Example: Response time, uptime percentage, error rate.
   - Use: Helps track how well a service is performing.
4. **Error Budget** - The allowable margin of failure within an SLO.
   - Example: If the SLO is 99.9% uptime, the error budget is 0.1% downtime (about 43.8 minutes per month).
   - Use: Helps balance innovation and reliability.
   - Error budget defines the acceptable level of failure before SLO violations.
![image](https://github.com/user-attachments/assets/7b93fc84-a951-417f-8190-2863db298798)

![image](https://github.com/user-attachments/assets/44042a63-ef17-43e9-96d7-877fdd1d7b7c)

**Metrics :**
1. Uptime/Availability (e.g., 99.9% availability)
2. Response Time (e.g., incidents responded to within 15 minutes)
3. Resolution Time (e.g., critical issues fixed within 4 hours)
4. Performance Metrics (e.g., API response time < 200ms)
5. Error Rates (e.g., < 0.01% failure rate)
6. Support & Escalation Process


#### API
An API (**Application Programming Interface**) is a set of rules and protocols that allows different software applications to communicate with each other. It defines how requests and responses should be structured, enabling seamless interaction between systems, applications, or services.
1. **General Explanation**
     - Facilitates communication between different software systems.
     - Enables modular and scalable development.
     - Promotes reusability of functionalities.
2. **For Developers (Backend, Frontend, Full Stack, Mobile, etc.)**
     - Backend Dev: Helps expose business logic securely to frontend applications.
     - Frontend Dev: Fetches and displays data dynamically using APIs.
     - Mobile Dev: Allows mobile apps to interact with cloud-based services.
3. **For System Architects & DevOps**
     - Helps integrate microservices for scalable applications.
     - Ensures interoperability between different platforms (e.g., cloud services).
4. **For Data Engineers & AI Engineers**
     - Enables fetching and sending large-scale data efficiently.
     - Provides access to external AI models and services.
- **Stages in API Gateway Deployment**
     - When creating an API Gateway (AWS API Gateway, Kong, Apigee, or any other API management tool), multiple stages allow for controlled deployment, testing, and versioning of APIs.
- **Common Stages in API Gateway**
     - **dev**	Development stage (used by developers, may have debugging enabled)
     - **test**	Used for API testing (integration and unit tests)
     - **staging (pre-prod)**	Pre-production stage, mirrors production for final validation
     - **prod**	Live production API, used by real customers
     - **beta**	Used for limited user testing before full production release
- **How SRE & DevOps Use API Gateway Stages**
     - **Traffic Routing** → Different environments for controlled API releases
     - **Rate Limiting & Throttling** → More relaxed in dev, strict in prod
     - **Logging & Monitoring** → More detailed logs in test/dev, essential metrics in prod
     - **Feature Flags** → Enable/disable API features per stage
     - **Rollback Mechanisms** → Deploy a stable previous version in case of issues
- **Canary Deployment & Blue-Green Deployment**
     - **Canary Release** → Gradually roll out new API versions to a small % of users before full deployment.
     - **Blue-Green Deployment** → Switch traffic between old (blue) and new (green) versions instantly.

#### REST API
**Representational State Transfer Application Programming Interface**
- its a web service which follows RESTful principles, it enables communication between client and server over HTTP.
     - **Stateless:** Each request is independent and does not store client session data.
     - **Client-Server Architecture:** Separation of concerns between UI (client) and backend (server).
     - **Cacheable:** Responses can be cached to improve performance.
     - **Uniform Interface:** Uses standard HTTP methods (GET, POST, PUT, DELETE).
     - **Resource-Based:** Data is represented as resources (e.g., /users, /orders).

#### API Key
- its an unique identifier used to authenticate requests made to an API. It helps control access and track API usage.
     - Client sends a request with an API key in the header, URL, or request body.
     - API server validates the key against a database or authentication system.
     - If valid, the request is processed; otherwise, it's rejected.
- use cases
     - **Authentication & Access Control** – Verify who is making API requests.
     - Rate Limiting & Throttling – Prevent abuse by restricting request frequency.
     - Monitoring & Logging – Track API usage for security and billing.
     - Integrations & Automation – Used in CI/CD pipelines, monitoring tools, and cloud services

#### API access
- details
     - it refers to how services, users, and applications interact with APIs securely and efficiently.
     - we need to manage API authentication, authorization, security, and monitoring.
     - secure API access in a DevOps environment
     - Authentication – Use OAuth 2.0, JWT, API keys, AWS IAM Roles to verify identity.
     - Authorization – Implement RBAC (Role-Based Access Control) or ABAC (Attribute-Based Access Control).
     - Rate Limiting & Throttling – Prevent abuse using tools like AWS API Gateway, Kong, or Nginx.
     - Encryption – Use TLS/SSL for securing data in transit.
     - Logging & Monitoring – Track API requests via CloudWatch, Prometheus, ELK Stack, or Datadog.

#### API Gateway
     - its a central entry point for managing, securing, and routing API requests between clients and backend services.
     - It acts as a reverse proxy that controls traffic, performs authentication, and enforces security policies.

#### TLS (Transport Layer Security) and SSL (Secure Sockets Layer) 
- Content
     - TLS (Transport Layer Security) is a cryptographic protocol that secures data transmission between systems over the internet.
     - It ensures confidentiality, integrity, and authentication of data, preventing eavesdropping, tampering, and impersonation.
- TLS ensures:
     - Confidentiality (Data is encrypted and cannot be read by unauthorized parties).
     - Integrity (Data is not altered during transmission).
     - Authentication (Validates the identity of the communicating parties).
- SSL (Secure Sockets Layer) is a cryptographic protocol designed to secure communication over the internet by encrypting data between a client (e.g., browser, API, microservice) and a server
- It helps protect sensitive information from hackers, ensuring privacy, integrity, and authentication.

- SSL/TLS Used (Real-World Use Cases)
     - Websites & APIs → HTTPS ensures secure web traffic (NGINX, Apache, AWS Load Balancer, API Gateway).
     - Kubernetes & Microservices → Mutual TLS (mTLS) secures communication (Istio, Linkerd).
     - Cloud Security → AWS, Azure, and GCP enforce TLS for data encryption.
     - DevOps Pipelines → TLS secures GitHub, Jenkins, CI/CD data transfers.
     - Database Encryption → TLS protects connections in MySQL, PostgreSQL, MongoDB.

#### Application Gateway
- its one of the most powerful Layer 7 load balancers in Azure, especially when combined with AKS for secure ingress traffic. It provides SSL termination, URL-based routing, WAF, and host-based routing, making it a go-to choice for secure, scalable applications.

#### 7 Layers

|Layer|Name|Example Protocols & Devices|
|-|-|-|
|L7|Application Layer|HTTP, HTTPS, FTP, DNS, SMTP|
|L6|Presentation Layer|SSL/TLS, Encryption, ASCII|
|L5|Session Layer|RPC, NetBIOS, PPTP, SMB|
|L4|Transport Layer|TCP, UDP, QUIC|
|L3|Network Layer|IP, ICMP, BGP, OSPF, Routers|
|L2|Data Link Layer|Ethernet, MAC, VLAN, ARP|
|L1|Physical Layer|Cables, Hubs, Fiber Optics|

## All ports information

|Port|Protocol|Usage|
| - | - |- |
|20, 21|FTP|File Transfer Protocol (Data & Control)|
|22|SSH|Secure Shell for remote access|
|23|Telnet|Remote login (Insecure, avoid using)|
|25|SMTP|Sending email (Restricted in AWS & Azure for spam control)|
|53|DNS|Domain Name System for name resolution|
|67, 68|DHCP|Dynamic Host Configuration Protocol (IP allocation)|
|80|HTTP|Unencrypted web traffic|
|110|POP3|Email retrieval|
|123|NTP|Network Time Protocol (Time synchronization)|
|143|IMAP|Email retrieval|
|443|HTTPS|Secure web traffic (SSL/TLS)|
|465, 587|SMTP (SSL/TLS)|Secure email sending|
|989, 990|FTPS|Secure FTP|
|3389|RDP|Remote Desktop Protocol (Windows)|
| - | **Cloud** |- |
|1194|OpenVPN|VPN connections|
|500, 4500|IPsec VPN|Used for site-to-site VPN tunnels (AWS VPN, Azure VPN Gateway)|
|1723|PPTP VPN|Point-to-Point Tunneling Protocol (Legacy VPN)|
|8443|HTTPS Alternative|Web services and APIs|
|8080|HTTP Alternate|Used by some web applications|
|3306|MySQL|MySQL Database connections|
|1433|MSSQL|Microsoft SQL Server|
|1521|Oracle DB|Oracle Database connections|
|27017|MongoDB|NoSQL database connections|
| - | **AWS** |- |
|2049|AWS EFS|Network File System (NFS) for Amazon EFS|
|6379|Amazon ElastiCache (Redis)|Redis database|
|8081|AWS CodeArtifact|AWS Package Manager|
|8983|Amazon OpenSearch|Elasticsearch-based managed service|
|9090|Prometheus|Monitoring and metrics|
| - | **Azure** |- |
|1433|Azure SQL|Azure SQL Database|
|5671, 5672|Azure Service Bus|Message queueing|
|6380|Azure Redis|Secure Redis Cache|
|1883, 8883|IoT Hub (MQTT)|IoT device messaging|
| _ | **Kubernetes** |- |
|10250|Kubelet API|Secure communication between nodes and master|
|10255|Kubelet Read-Only API|Deprecated but still used in some cases|
|6443|Kubernetes API Server|Cluster control plane communication|
|2379-2380|etcd|Kubernetes cluster state storage|
|30000-32767|NodePort|Kubernetes service exposing apps externally|
|-  | **Monitoring** |- |
|9200|Elasticsearch|Log aggregation|
|1514, 514|Syslog|System logging|
|5601|Kibana|Elasticsearch Dashboard|
|8086|InfluxDB|Time-series database|

#### Critical service is unresponsive (HTTP 500 errors spike).
     - Alert fires in Prometheus Alertmanager.
     - Lambda function triggers a Kubernetes job that restarts pods.
     - If issue persists, an auto-remediation script runs a rollback via ArgoCD.
     - On-call SRE is notified via alert messaging

#### software versions
- version
     - Docker: 20.10.17 (released February 15, 2022)
     - Kubernetes: 1.25.0 (released August 31, 2022)
     - Jenkins: 2.361 (released February 8, 2023)
     - GitLab CI/CD: 15.0 (released May 22, 2022)
     - CircleCI: 2.1 (released November 15, 2022)
     - Git: 2.39.1 (released February 13, 2023)
     - Prometheus: 2.39.1 (released February 15, 2023)
     - Grafana: 9.3.6 (released February 14, 2023)
     - ELK Stack (Elasticsearch, Logstash, Kibana): 8.6.0 (released February 1, 2023)
     - Terraform: 1.3.7 (released February 16, 2023)
     - Ansible: 2.14.1 (released February 7, 2023)

#### HTTP Error Codes

|Error Code|Category|Meaning|Possible Cause|
|-|-|-|-|
|400|Client Error (4xx)|Bad Request|Invalid input, malformed request|
|401|Client Error (4xx)|Unauthorized|Authentication required but not provided|
|403|Client Error (4xx)|Forbidden|User does not have permission|
|404|Client Error (4xx)|Not Found|Resource does not exist|
|405|Client Error (4xx)|Method Not Allowed|HTTP method (GET, POST, etc.) is incorrect|
|408|Client Error (4xx)|Request Timeout|Client took too long to send a request|
|429|Client Error (4xx)|Too Many Requests|Rate limiting exceeded|
| 500 | Server Error (5xx) | Internal Server Error | Generic server failure | 
| 502 | Server Error (5xx) | Bad Gateway | Invalid response from upstream server | 
| 503 | Server Error (5xx) | Service Unavailable | Server is overloaded or under maintenance | 
| 504 | Server Error (5xx) | Gateway Timeout | Timeout waiting for a response from another service |

#### Observability
- Observability is the ability to measure, analyze, and troubleshoot a system’s performance by collecting and analyzing metrics, logs, and traces. In DevOps & SRE, observability ensures system reliability, proactive monitoring, and rapid incident response.
     - Observability is the ability to measure system behavior through metrics, logs, and traces.
     - Monitoring is about predefined alerts and thresholds for known issues.

- Three Pillars of Observability - **Metrics / Logs / Traces**

|Pillar|Definition|Examples|
|-|-|-|
|**Metrics**|performance data (CPU, memory, latency, error rates)|Prometheus, CloudWatch, Datadog|
|**Logs**|system activity (application logs, system logs)|ELK Stack, Splunk, Loki|
|**Traces**|End-to-end request tracking across microservices|Jaeger, AWS X-Ray, OpenTelemetry|
|Visualization| Raw metrics, logs, traces into dashboards |Grafana, Kibana|
|Alerting| we get notified about system |Alertmanager, PagerDuty, VictorOps|

- log management in observability
     - ELK Stack (Elasticsearch, Logstash, Kibana) for log aggregation and search.
     - Grafana Loki for lightweight log storage.
     - Splunk for enterprise-scale log analysis.

#### Metrics to Monitor in Observability
- Contents
     - Infrastructure Metrics
     - Application Performance Metrics
     - Database Metrics
     - Security Metrics
     - Network & API Metrics
     - Kubernetes Metrics
     - Log-Based Metrics
     - Business & SLA Metrics

1. Infrastructure Metrics
     - CPU Usage (%)
     - Memory Usage (%)
     - Disk I/O (Read/Write Speed, Latency)
     - Disk Usage (%)
     - Network Bandwidth (Inbound/Outbound Traffic)
     - Network Packet Loss & ErrorsSystem Load Average
2. Kubernetes Metrics
     - Pod CPU & Memory Usage
     - Pod Restart Count
     - Container Uptime & Status
     - Node Resource Utilization
     - Kube API Server Latency & Request Rate
     - Kubernetes Scheduler Performance
     - Kubernetes etcd Health & Latency
3. Application Performance Metrics
     - Request Latency (p50, p95, p99 percentiles)
     - Request Throughput (Requests per Second - RPS)
     - HTTP Status Codes (2xx, 4xx, 5xx errors)
     - Database Query Latency
     - Cache Hit Ratio
     - Thread/Connection Pool Utilization
4. Database Metrics
     - Query Execution Time
     - Active Connections
     - Replication Lag
     - Cache Miss Ratio
     - Database Disk Usage
     - Transaction Commit/Rollback Count
5. Security Metrics
     - Unauthorized Access Attempts
     - IAM Policy Changes
     - SSL/TLS Certificate Expiry
     - API Gateway Unauthorized Errors
     - Unusual Login Patterns (Anomaly Detection)
6. Network & API Metrics
     - API Request Success & Failure Rate
     - DNS Resolution Time
     - Load Balancer Connection Count
     - Rate of 4xx and 5xx Errors
     - TCP Retransmissions & Packet Drops
7. Log-Based Metrics
     - Error Rate (Number of ERROR Logs/Min)
     - Application Crash Frequency
     - Slow Query Logs Count
     - Critical System Events (Service Restart, Failures)
8. Business & SLA Metrics
     - Uptime (SLA Compliance %)
     - SLO (Service Level Objective) Violations
     - Customer Transactions Per Second
     - Revenue Impacting Errors

#### Prometheus – Metrics & Alerting
- Best For: Kubernetes, Cloud Monitoring, Infrastructure Metrics
- Type: Time-Series Database (TSDB)
- Key Features
     - Pull-based metrics collection using HTTP endpoints (/metrics).
     - Powerful query language (PromQL) for real-time analysis.
     - Service discovery for Kubernetes, EC2, Docker, etc.
     - Alerting with Alertmanager (Slack, PagerDuty, Email).
- Common Prometheus Metrics to Monitor
     - Infrastructure: CPU, memory, disk I/O, network bandwidth
     - Kubernetes: Pod health, node resource usage, container restarts
     - Application: Request latency (p50, p95, p99), error rates

#### Datadog – Full-Stack Monitoring & Observability
- Best For: Multi-cloud, AIOps, Advanced Tracing
- Type: SaaS-based Monitoring & APM
- Key Features
     - Auto-instrumentation for logs, metrics, traces (APM).
     - Real-time dashboards with AI-driven insights.
     - Log management with anomaly detection.
     - Integrations: AWS, Azure, Kubernetes, Docker, MySQL, Redis.
- Common Datadog Monitors
     - Infrastructure: Host metrics (CPU, Memory, Disk, Network).
     - APM (Application Performance Monitoring): Request latency, DB queries.
     - Security Monitoring: Unauthorized access attempts, log anomalies.

#### ELK Stack (Elasticsearch, Logstash, Kibana) – Log Analytics
- Best For: Log Aggregation, Real-time Log Analysis
- Type: Open-source Log Management
- Key Features
     - Log ingestion from multiple sources (Logstash, Beats, Fluentd).
     - Powerful search with Elasticsearch.
     - Visualization with Kibana dashboards.
     - Machine learning for anomaly detection.
- Common Log Queries in Kibana
     - Find All 500 Errors
     - Monitor Kubernetes Pod Failures
     - Detect Unauthorized Access
 
Kafka Architecture
- Apache Kafka is a distributed event streaming platform designed for high throughput, scalability, and fault tolerance.
- It follows a publish-subscribe model and is widely used for real-time data streaming, event-driven architectures, log processing.
- **Producers**
     - Producers write (publish) data to Kafka topics.
     - They push messages to a broker, which assigns them to a partition.
     - Key Features:
          - Asynchronous processing – Reduces latency.
          - Partitioning logic – Messages can be partitioned by key.
          - Acks configuration for reliability (acks=0, acks=1, acks=all).
- **Kafka Broker (Server)**
     - A broker is a Kafka server responsible for storing, receiving, and serving messages.
     - Kafka clusters consist of multiple brokers.
     - Each broker handles multiple partitions of different topics.
     - Key Features:
          - Scalability – New brokers can be added dynamically.
          - Fault Tolerance – Replication ensures data durability.
          - Leader Election – One broker is assigned as the partition leader; others are followers.
- **Partitions & Replication**
     - Kafka splits topics into partitions to achieve parallelism.
     - Each partition is stored in multiple brokers using replication (replication.factor).
     - Leader-Follower Model:
          - Leader partition handles all read/writes.
          - Followers replicate data and take over if the leader fails.
     - ISR (In-Sync Replicas):
          - Brokers that have fully replicated the leader’s data.
          - Messages are committed only when written to min.insync.replicas.
- **Consumers & Consumer Groups**
     - Consumers read messages from Kafka topics.
     - Each consumer belongs to a consumer group.
     - Kafka ensures each partition is consumed by only one consumer within a group.
     - Key Concepts:
          - Consumer Offset Management: Kafka keeps track of the last read message using offsets (stored in Kafka or an external store like Zookeeper).
          - Rebalancing: If a consumer joins/leaves a group, Kafka reassigns partitions.
          - Parallel Processing: Multiple consumers can process different partitions simultaneously.
- **Zookeeper** (Metadata Management)
     - Kafka (before version 2.8) relies on Zookeeper for:
          - Leader election (chooses partition leaders).
          - Broker discovery & failure detection.
          - Topic and partition metadata storage.
          - Consumer group coordination.

#### ArgoCD Architecture
- its a declarative, GitOps-based continuous delivery (CD) tool for Kubernetes. It automates application deployment, syncing, and rollback in a Kubernetes environment.
     - GitOps-based deployment (declarative & version-controlled).
     - Automatic syncing of applications from Git repositories.
     - Supports Helm, Kustomize, Jsonnet, and raw YAML.
     - Provides a GUI, CLI, and API for managing deployments.
     - Supports multi-cluster deployments from a single ArgoCD instance.
     - Self-healing: Detects drift and automatically restores the desired state
- **components**:
     - **API Server**
          - Provides a UI, CLI, and API for managing applications.
          - Handles authentication & authorization.
     - **Repository Server**
          - Fetches application manifests from Git repositories.
          - Supports Helm charts, Kustomize overlays, Jsonnet, and raw YAML files.
     - **Application Controller**
          - Monitors the desired state from Git and compares it to the cluster’s actual state.
          - Syncs applications by applying Kubernetes manifests.
          - Triggers rollbacks if a deployment fails.
     - **Dex (Optional)**
          - Handles Single Sign-On (SSO) authentication (OAuth, LDAP, GitHub, SAML).
            
#### GitOps Works
- work flow
     1. Developers push changes to Git - Code, configuration, and Kubernetes manifests are stored in a Git repository.
     2. GitOps Operator (ArgoCD/FluxCD) detects the change - The GitOps tool monitors the repo and detects changes automatically.
     3. GitOps tool applies changes to the cluster - ArgoCD/FluxCD syncs the new state to Kubernetes.
     4. Continuous Reconciliation - If someone modifies a Kubernetes resource manually, GitOps detects drift and reverts it.
- Best Practices for GitOps
     - **Separate Repositories:** Maintain different repos for app code and infrastructure.
     - **Use Branching Strategies:** Use feature branches and PR approvals for deployments.
     - **Implement RBAC & Access Control:** Restrict who can modify Git repositories.
     - **Monitor GitOps Pipelines:** Use Prometheus, Loki, and Grafana to track deployments.
     - **Automate Secret Management:** Use Sealed Secrets, HashiCorp Vault, or AWS Secrets Manager instead of committing secrets to Git.

#### Cassandra architecture
- Apache Cassandra is a distributed NoSQL database designed for high availability, fault tolerance, and scalability. Key architectural principles include:
     - Peer-to-Peer Architecture: No master/slave concept; all nodes are equal.
     - Partitioned Data Model: Data is distributed across multiple nodes using a consistent hashing-based partitioning strategy.
     - Replication & Consistency: Uses tunable consistency with quorum-based reads and writes.
     - Eventual Consistency: Ensures high availability by using hinted handoffs and gossip protocol for synchronization.
     - SSTable-Based Storage Engine: Uses Log-Structured Merge (LSM) Trees for efficient writes and compaction.
     - Multi-DC Support: Can replicate data across multiple data centers for disaster recovery (DR) and fault tolerance.

#### Cassandra handle high availability and fault tolerance
- Cassandra ensures high availability and fault tolerance through:
     -  Data Replication: Uses replication factor (RF) to store multiple copies of data across nodes.
     -  Consistent Hashing: Distributes data evenly across nodes to prevent hotspots.
     -  Gossip Protocol: Nodes share state information to detect failures.
     -  Hinted Handoff: Temporarily stores writes for unreachable nodes and delivers them once the node recovers.
     -  Read Repair: Ensures consistency by reconciling outdated data during reads.
     -  Multi-Datacenter Replication: Supports global deployments with low-latency, geo-distributed clusters.

#### Cassandra’s different consistency levels, how do they work
- Cassandra provides tunable consistency levels for reads and writes, balancing between strong consistency and availability:
- Write Consistency Levels:
     - ANY – The fastest; data is written to at least one node (even hinted handoff).
     - ONE – Acknowledged by at least one replica node.
     - QUORUM – Acknowledged by (RF/2 + 1) replicas (best balance of consistency and availability).
     - ALL – Written to all replica nodes (strong consistency but high latency).
- Read Consistency Levels:
     - ONE – Reads from a single replica.
     - QUORUM – Reads from (RF/2 + 1) nodes to ensure consistency.
     - ALL – Reads from all replicas, ensuring the latest data.
     - LOCAL_QUORUM – Reads from a quorum of local datacenter nodes (optimized for multi-DC setups).

#### Cassandra handle automatic failover and node recovery
- Cassandra automatically handles failures and recovery using:
     -  Gossip Protocol: Detects node failures by continuously exchanging state information.
     -  Hinted Handoff: If a node is down, another node temporarily stores its writes and delivers them later.
     -  Read Repair: If stale data is found during a read, the latest version is written back.
     -  Anti-Entropy Repair (nodetool repair): Syncs missing data between nodes to prevent inconsistencies.
     -  Dynamic Snitching: Redirects queries to the most responsive nodes, optimizing performance.
     -  Multi-DC Awareness: Failures in one region do not affect other regions due to multi-DC replication.

#### Cassandra - monitor and troubleshoot performance issues
- Cassandra’s performance monitoring focuses on latency, compaction, disk I/O, and replication delays.
     - Monitoring Metrics (Prometheus, Grafana, ELK, Datadog)
     - nodetool status → Check cluster health, node availability.
     - nodetool tpstats → Monitor thread pool usage (pending tasks).
     - nodetool cfstats → Identify slow queries and high tombstones.
     - nodetool compactionstats → Check for compaction issues.
- Prometheus Metrics:
     - cassandra_client_request_latency (High latency indicates bottlenecks).
     - cassandra_gc_time (Frequent GC pauses impact performance).
     - cassandra_pending_tasks (High values indicate overload).
     - Common Troubleshooting Steps
     - High Read Latency:
- Check SSTable compaction (nodetool compact to trigger manual compaction).
     - Optimize caching and partitioning strategies.
     - Increase heap size and tune garbage collection (GC tuning).
     - High Write Latency:
- Ensure enough free disk space.
     - Tune memtable flush thresholds to avoid frequent writes.
     - Check replication factor and consistency levels (QUORUM vs. ALL).
     - Node Down Issues:
            - Restart the node and check logs in /var/log/cassandra/system.log.
            - Run nodetool status to verify cluster topology.

#### Cassandra handle schema migrations
- Unlike traditional relational databases, Cassandra does not support ALTER TABLE efficiently due to its distributed nature.
- Best Practices for Schema Migrations:
     -  Use Versioned Schema Changes (Liquibase, Flyway):

- Store schema versions in Git and apply controlled updates.
     -  Rolling Updates:
- Add new columns but avoid dropping columns immediately (use soft deletes).
     -  Backfill Data:
- For structure changes, create a new table, migrate data, and deprecate the old one.
     -  Monitor Schema Changes:
- Use cqlsh -e "DESCRIBE SCHEMA" to validate changes.

#### Cassandra - optimize read and write performance
- answer
     - Write Performance Optimizations
     -  Use Leveled Compaction Strategy (LCS) for read-heavy workloads.
     -  Set memtable_flush_period_in_ms to control in-memory writes.
     -  Optimize batch writes but avoid large batches (>5k operations).

     - Read Performance Optimizations
     -  Store hot data in Key Caching (caching: KEYS_ONLY).
     -  Use Bloom Filters to speed up read path.
     -  Avoid wide partitions (keep partitions <100MB).
     -  Tune consistency levels (LOCAL_QUORUM for multi-DC).

#### Cassandra difference between Compaction and Repair
-  Compaction: Merges SSTables, reducing disk usage and improving read efficiency.
     - Types: SizeTiered (default), Leveled (LCS), TimeWindow (TWCS).
     - Triggered automatically or manually (nodetool compact).
     -  Repair: Synchronizes data across replicas to maintain consistency.
     - Essential in multi-DC clusters.
     - Triggered using nodetool repair.
     - Should be run periodically to prevent inconsistencies.

#### Cassandra protect sensitive data cluster on AWS, 
- implement:
     - Encryption (at rest & in transit, AWS KMS, SSL/TLS)
     - Access Control (RBAC, IAM policies, AWS PrivateLink)
     - Network Security (VPC, Security Groups, Private Subnets)
     - Data Masking & Redaction (Column-level encryption, masking)
     - Auditing & Compliance (Logging, IAM tracking, GDPR/HIPAA)
     - Secure Backup & Disaster Recovery (Encrypted backups, multi-region failover)

#### Deployment & Automation
- Infrastructure as Code (IaC):
     - Use Terraform, Ansible, or Helm to provision and manage Cassandra clusters.
     - Automate cluster scaling and node additions using Terraform modules.
- CI/CD for Schema Changes:
     - Implement Liquibase or Flyway for controlled schema migrations.
     - Automate schema versioning to avoid downtime and conflicts.
- Containerization & Orchestration:
     - Deploy Cassandra using Docker & Kubernetes with StatefulSets.
     - Automate persistent storage management using K8s CSI (Container Storage Interface).

#### Performance Tuning & Optimization
- Query Optimization:
     - Analyze slow queries using nodetool toppartitions and optimize queries based on read/write patterns.
Compaction & Repair Strategies:
     - Use Leveled Compaction Strategy (LCS) for read-heavy workloads.
     - Automate nodetool repair using cron jobs or Kubernetes Jobs to maintain consistency.
- Resource Allocation & Tuning:
     - Optimize JVM settings (heap size, garbage collection tuning).
     - Set appropriate replication factor and consistency levels based on SLAs.

#### Cassandra Monitoring & Observability
- Metric Collection & Logging:
     - Use Prometheus, Grafana, Datadog, or New Relic for monitoring Cassandra metrics (latency, read/write throughput, GC pauses).
     - Enable structured logging using Loki, ELK (Elasticsearch, Logstash, Kibana), or Splunk.
- Alerting & Incident Response:
     - Set up alerts for high disk usage, node failures, high latency, tombstone warnings, and replication lag.
     - Implement self-healing automation to replace unhealthy nodes.
- Distributed Tracing:
     - Use Jaeger or OpenTelemetry for query-level distributed tracing.

#### Cassandra Backup & Disaster Recovery
- Automated Backup Strategies:
     - Schedule incremental & full snapshots using nodetool snapshot and store in AWS S3/Azure Blob/GCS.
     - Implement Point-in-Time Recovery (PITR) using Medusa for Cassandra.
- Replication & Failover:
     - Use Multi-DC replication for high availability and failover support.
     - Automate failover handling via Service Mesh (Istio/Linkerd) or load balancers.
- Testing Restore Processes:
     - Regularly test disaster recovery plans using automated scripts.

#### Cassandra Security & Compliance
- Access Management:
     - Implement Role-Based Access Control (RBAC) using cassandra.auth_roles.
     - Use Vault or AWS KMS for secrets management (passwords, tokens, and certificates).
- Encryption & Data Protection:
     - Enable encryption-at-rest (LUKS, TDE) and encryption-in-transit (TLS).
- Auditing & Compliance:
     - Maintain audit logs for query execution and access patterns.
     - Ensure compliance with GDPR, HIPAA, or PCI-DSS.

#### Incident Response & RCA
- Fault Tolerance & Auto-healing:
     - Implement self-healing mechanisms to restart failed nodes.
     - Use Chaos Engineering (Gremlin, LitmusChaos) to simulate failures and improve resilience.
- Root Cause Analysis (RCA):
     - Use logs, metrics, and traces for post-incident analysis.
     - Implement blameless postmortems to document lessons learned.

#### Cassandra Cost Optimization
- Efficient Resource Usage:
     - Optimize storage usage by avoiding unnecessary tombstones and ensuring proper compaction.
     - Use auto-scaling for cloud-based Cassandra deployments.
- Spot Instances & Savings Plans:
     - Run non-critical workloads on spot instances to reduce costs.
     - Optimize Cassandra read/write consistency to balance between performance and cost.

1. Pre-Upgrade Planning & Assessment
- Before starting, gather key details about the current Cassandra cluster:
- Check Current Version & Compatibility
     - Identify the current Cassandra version (cassandra -v or nodetool version).
     - Read the release notes for the target version.
     - Validate if the upgrade is supported directly (Cassandra supports only one major version jump at a time).
- Backup & Disaster Recovery Plan
     - Take a full snapshot of each node using:
          - nodetool snapshot -t pre-upgrade-backup
          - Copy backup files to a remote storage (AWS S3, Azure Blob, GCS).
     - Ensure schema backups:
          - cqlsh -e "DESC SCHEMA" > schema-backup.cql
- Pre-Upgrade Health Check
     - Check cluster status:
          - nodetool status
     - Check for pending repairs:
          - nodetool repair
     - Ensure no tombstone issues:
          - nodetool compactionstats
     - Validate enough free disk space (at least 50% of disk free for major version upgrades).
2. Upgrade Strategy - Rolling Upgrade Approach
- Cassandra allows a rolling upgrade, meaning nodes are upgraded one at a time while keeping the cluster operational.
     - Upgrade Order:
          - Seed nodes (one by one)
          - Non-seed nodes
          - Finalizing & testing
3. Upgrade Process (Node-by-Node Rolling Upgrade)
- Step 1: Disable Automatic Processes
     - Before upgrading a node:
     - nodetool drain
     - Draining flushes memtables and stops accepting writes to prevent data loss.
     - Stop the Cassandra service:
          - systemctl stop cassandra  # Linux
          - service cassandra stop    # Older Linux
- Step 2: Upgrade the Cassandra Binary
     - Option 1: Using Package Manager
          - If using Debian/Ubuntu:
          - sudo apt update && sudo apt upgrade cassandra
          - If using RHEL/CentOS:
          - sudo yum update cassandra
     - Option 2: Manual Upgrade
          - Download the new Cassandra tarball:
          - wget https://downloads.apache.org/cassandra/X.Y.Z/apache-cassandra-X.Y.Z-bin.tar.gz
          - Extract and replace the old version:
          - tar -xvf apache-cassandra-X.Y.Z-bin.tar.gz
- Step 3: Update Configuration Files
     - Compare and update cassandra.yaml and jvm.options.
     - Ensure the cluster name, seed nodes, and replication settings are intact.
     - If upgrading a major version, check if schema changes are required.
- Step 4: Restart & Validate Node
     - Restart Cassandra:
          - systemctl start cassandra
     - Check if the node is back in the cluster:
          - nodetool status
     - Verify logs for errors:
          - cat /var/log/cassandra/system.log | grep ERROR
- Step 5: Run Post-Upgrade Tests
     - After upgrading all nodes:

#### nodetool repair:
- nodetool repair
     - Verify data consistency using nodetool status and check for missing data:
- nodetool validate
     - Test application queries using a sample workload.
- Rollback Plan (If Upgrade Fails)
- If the upgrade introduces issues:
     - Stop Cassandra
     - systemctl stop cassandra
     - Restore from backup
     - rm -rf /var/lib/cassandra/*
     - cp -r /backup/cassandra/* /var/lib/cassandra/
     - Restart the old version
     - systemctl start cassandra

#### Protecting Sensitive Data in a Cassandra Cluster
- Encryption for Data Protection
     - Encryption at Rest (EBS, S3, TDE)
     - Identity & Access Management (IAM, RBAC)
     - Secure Networking (VPC, Security Groups, Private Endpoints)
     - Data Masking & Redaction
     - Auditing & Compliance (Logging, Monitoring, IAM Policies)
     - Backup & Disaster Recovery (AWS S3, KMS, PITR)

- protect sensitive data in a Cassandra cluster on AWS, implement:
     - Encryption (at rest & in transit, AWS KMS, SSL/TLS)
     - Access Control (RBAC, IAM policies, AWS PrivateLink)
     - Network Security (VPC, Security Groups, Private Subnets)
     - Data Masking & Redaction (Column-level encryption, masking)
     - Auditing & Compliance (Logging, IAM tracking, GDPR/HIPAA)
     - Secure Backup & Disaster Recovery (Encrypted backups, multi-region failover)

#### Ansible
- I used Ansible to automate server configurations, application deployments, and infrastructure provisioning across cloud and on-premises environments

1. **Core Configuration File** 
     - **ansible.cfg** → Configures Ansible behavior (e.g., inventory path, SSH settings)
2. **Inventory Files**
     - inventory.ini → Static inventory in INI format
     - inventory.yaml → Inventory in YAML format
     - dynamic_inventory.py → Custom script for dynamic inventory
3. **Playbook Files**
- main.yml or site.yml → Main playbook file defining tasks
- Other playbooks (deploy.yml, setup.yml, etc.) can be created for different purposes.
4. **Role Structure (Inside roles/ Directory**)
     - Each role contains the following files:
     - tasks/main.yml → Defines tasks for the role
     - handlers/main.yml → Defines event-driven handlers
     - vars/main.yml → Role-specific variables
     - defaults/main.yml → Default variables for the role
     - meta/main.yml → Role metadata (dependencies, author info)
     - templates/*.j2 → Jinja2 template files (e.g., nginx.conf.j2)
     - files/* → Static files to be copied to remote hosts
     - tests/* → Test cases for the role
5. **Group and Host Variables**
     - group_vars/all.yml → Variables for all hosts
     - group_vars/webservers.yml → Variables for a specific group
     - host_vars/host1.yml → Variables for a specific host
6. **Modules and Plugins**
     - library/custom_module.py → Custom Python module
     - filter_plugins/custom_filter.py → Custom Jinja2 filter
7. **Other Supporting Files**
     - requirements.yml → Defines dependencies (for ansible-galaxy)
     - secrets.yml → Encrypted variables (managed with ansible-vault)
     - custom_script.sh → Bash scripts used by Ansible
     - logfile.log → Log file if logging is enabled

#### Ansible file structure

```
ansible-project/
│-- inventory/              # Inventory files (host groups)
│   ├── production          # Production environment inventory
│   ├── staging             # Staging environment inventory
│
│-- group_vars/             # Variables for groups of hosts
│   ├── all.yml             # Variables for all hosts
│   ├── webservers.yml      # Variables for webservers group
│
│-- host_vars/              # Variables for individual hosts
│   ├── server1.yml         # Variables for server1
│   ├── server2.yml         # Variables for server2
│
│-- roles/                  # Role-based automation structure
│   ├── common/             # Example role: common configurations
│   │   ├── tasks/          # Tasks for the role
│   │   │   ├── main.yml
│   │   ├── handlers/       # Handlers (like service restarts)
│   │   │   ├── main.yml
│   │   ├── templates/      # Jinja2 templates for config files
│   │   ├── files/          # Static files to be copied
│   │   ├── vars/           # Role-specific variables
│   │   ├── defaults/       # Default variables (lower precedence)
│   │   ├── meta/           # Role metadata (dependencies)
│   │   ├── tests/          # Role tests
│   ├── webserver/          # Another role: webserver setup
│
│-- playbooks/              # Playbooks directory
│   ├── site.yml            # Main playbook
│   ├── webservers.yml      # Playbook for webservers
│
│-- ansible.cfg             # Ansible configuration file
│-- requirements.yml        # Role dependencies
│-- README.md               # Documentation
```

#### Ansible Key Components
```
     - **inventory/:** - Defines target hosts or groups; usually stored in inventory/hosts or as a dynamic script.
     - **Modules:** - Reusable units of code that perform specific tasks like installing packages or copying files.
     - **roles/:** Structured collections of tasks, variables, templates, handlers used to organize and reuse Ansible code.
     - **playbooks/:** - YAML files that define automation workflows using plays, targeting groups of hosts with specific tasks. stored at the root or in playbooks/ directory.
     - **Variables** – Key-value data used in playbooks; stored in vars/ or group_vars/host_vars/ directories.
     - **group_vars/ & host_vars/:** - Store variables for groups and individual hosts.
     - **Tasks:** - Individual units of work in a playbook that call modules to perform actions on target hosts.
     - **Templates:** - Jinja2-based files used to dynamically generate configuration files or scripts during playbook execution.
     - **Handlers:** - Special tasks triggered only when notified by other tasks, typically used for actions like restarting services.
     - **ansible.cfg:** - Ansible configuration settings.
     - **requirements.yml:** - Defines external roles dependencies (useful for ansible-galaxy)
     - **Plugins:** - Extend Ansible's core functionality (e.g., connection, callback, lookup, etc.).
     - **Collections** – Distributable bundles of roles, modules, plugins, etc.; stored under collections/ or pulled via ansible-galaxy.
     - **Facts:** - System information automatically gathered from managed hosts, such as IP addresses or OS details. ( ansible_facts or setup module.)
     - **Vault:** - its a feature for encrypting sensitive data like passwords or keys within Ansible files. any YAML file encrypted via ansible-vault (e.g., secrets.yml).
```

#### Ansible use cases
- use cases
     - Server Hardening (Security Compliance) - we can secure servers by disabling root login, updating packages
     - Deploying a Web Server - Install and start an Nginx web server
     - CI/CD Pipeline Integration - Deploy an application from GitHub using Ansible
     - Running ad-hoc commands
     - Managing users

#### Playbook sample
- We write YAML files where we can define tasks to be executed on remote servers.
- which contain plays (groups of tasks), specify hosts, tasks, handlers, variables, and roles.
- Action
     - create a patch-linux.yml  and add the steps like upgrade, name, when, yum, apt, reboot
     - Update the ( hosts.ini ) where host information picked from inventory file
     - **ansible-playbook -i hosts.ini patch-linux.yml**
```
[linux-servers]
web01 ansible_host=192.168.1.10
db01 ansible_host=192.168.1.11
app01 ansible_host=192.168.1.12

[linux-servers:vars]
ansible_user=ansibleadmin
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

```
---
- name: Patch multiple Linux servers
  hosts: all
  become: yes
  gather_facts: yes
  tasks:
    - name: Update apt cache (for Debian/Ubuntu)
      apt:
        update_cache: yes
      when: ansible_os_family == "Debian"
    - name: Install security and regular updates (Debian/Ubuntu)
      apt:
        upgrade: dist
      when: ansible_os_family == "Debian"
    - name: Update yum cache (for RHEL/CentOS/AlmaLinux/Rocky)
      yum:
        update_cache: yes
      when: ansible_os_family == "RedHat"
    - name: Apply all updates (RHEL/CentOS/AlmaLinux/Rocky)
      yum:
        name: '*'
        state: latest
      when: ansible_os_family == "RedHat"
    - name: Reboot if needed (optional)
      reboot:
        reboot_timeout: 600
        pre_reboot_delay: 10
        post_reboot_delay: 30
        test_command: uptime
      when: ansible_facts['os_family'] in ['RedHat', 'Debian']
```

#### Playbook: Install/Uninstall Agent (Generic Template)
- agent_manage.yml
- hosts.ini
- Commands
     - ansible-playbook -i hosts.ini agent_manage.yml -e "agent_action=install agent_package=zabbix-agent agent_service=zabbix-agent"
     - ansible-playbook -i hosts.ini agent_manage.yml -e "agent_action=uninstall agent_package=zabbix-agent agent_service=zabbix-agent"

```
---
- name: Install or uninstall monitoring/logging agent
  hosts: all
  become: yes
  vars:
    agent_action: install   # change to 'uninstall' to remove the agent
    agent_package: myagent
    agent_service: myagent
  tasks:
    - name: Install agent (Debian/Ubuntu)
      apt:
        name: "{{ agent_package }}"
        state: present
      when:
        - ansible_os_family == "Debian"
        - agent_action == "install"
    - name: Install agent (RHEL/CentOS)
      yum:
        name: "{{ agent_package }}"
        state: present
      when:
        - ansible_os_family == "RedHat"
        - agent_action == "install"
    - name: Uninstall agent (Debian/Ubuntu)
      apt:
        name: "{{ agent_package }}"
        state: absent
      when:
        - ansible_os_family == "Debian"
        - agent_action == "uninstall"
    - name: Uninstall agent (RHEL/CentOS)
      yum:
        name: "{{ agent_package }}"
        state: absent
      when:
        - ansible_os_family == "RedHat"
        - agent_action == "uninstall"
    - name: Ensure agent service is started and enabled
      systemd:
        name: "{{ agent_service }}"
        enabled: yes
        state: started
      when: agent_action == "install"
    - name: Ensure agent service is stopped and disabled
      systemd:
        name: "{{ agent_service }}"
        enabled: no
        state: stopped
      when: agent_action == "uninstall"
```

#### Python file structure

```
python-project/
│-- src/                     # Source code directory (for apps/libraries)
│   ├── package_name/         # Main package
│   │   ├── __init__.py       # Marks directory as a package
│   │   ├── module1.py        # Module 1
│   │   ├── module2.py        # Module 2
│   │   ├── utils/            # Utility functions
│   │   │   ├── __init__.py
│   │   │   ├── helpers.py
││-- tests/                    # Unit & integration tests
│   ├── __init__.py
│   ├── test_module1.py
│   ├── test_module2.py
││-- scripts/                  # Executable scripts
│   ├── run.py                # Entry point for the application
│   ├── manage.py             # For admin tasks (if applicable)
││-- docs/                     # Documentation
│   ├── index.md              # Project overview
│   ├── usage.md              # Usage guide
││-- configs/                  # Configuration files
│   ├── config.yaml           # YAML configuration
│   ├── settings.json         # JSON settings
││-- .venv/                    # Virtual environment (optional)
│-- requirements.txt          # Dependencies list
│-- pyproject.toml            # Build system metadata (for modern projects)
│-- setup.py                  # Installation & packaging script
│-- README.md                 # Project documentation
│-- .gitignore                # Git ignore file
│-- .pre-commit-config.yaml   # Pre-commit hooks (optional)
```

#### Python Library

|Library|Purpose|
|-|-|
|boto3|AWS SDK for interacting with AWS services|
|requests|Making HTTP API calls|
|pandas|Data analysis and manipulation|
|numpy|Numerical computing|
|matplotlib/seaborn|Data visualization|
|scikit-learn|Machine learning algorithms|
|Flask/Django|Web application frameworks|
|SQLAlchemy|Database ORM (Object-Relational Mapping)|
|paramiko|SSH automation for remote servers|
|pytest|Unit testing framework|
|Celery|Asynchronous task scheduling|
|Twisted|Asynchronous networking|

#### Python use cases
-  Automating Cloud Resource Management (AWS EC2)
     - we can write a script stops idle EC2 instances to save costs.
-  Kubernetes Self-Healing (Restarting Failed Pods)
     - If a pod crashes, this script deletes it, allowing Kubernetes to recreate it.
-  CI/CD Pipeline Automation (Triggering Jenkins Job)
     - script triggers a Jenkins job, integrating Python into CI/CD pipelin
-  Automating Cloud Cost Optimization
     - Problem: Unused AWS resources were increasing costs.
     - Solution: Created a Python script using boto3 to detect and stop idle EC2 instances based on CPU usage and tags.
-  Kubernetes Self-Healing & Auto-Scaling
     - Problem: Kubernetes pods were crashing due to memory leaks, requiring manual intervention.
     - Solution: Used Python with kubernetes client to detect and restart failing pods automatically.
- CI/CD Pipeline Automation
     - Problem: Developers were manually triggering deployments, leading to inconsistencies.
	 - Solution: Wrote a Python script to trigger Jenkins pipelines automatically after a successful code merge in GitHub.
- Log Analysis & Anomaly Detection
     - Problem: Debugging application failures was time-consuming due to large log files.
	 - Solution: Built a Python script using elasticsearch-py to query logs and detect anomalies.

#### Prometheus & Grafana  -  monitoring and observability in cloud-native environments.
1. **Node Exporter**
     - Purpose: Collects system-level metrics from nodes (CPU, memory, disk, network, etc.).
     - How it Works: Runs as a daemon on each host, exposing metrics at /metrics endpoint.
     - Integration: Prometheus scrapes these metrics using prometheus.yml configuration.
     - Use Case: Essential for infrastructure monitoring in Kubernetes, VMs, and bare-metal servers.
2. **Batch Job** - Purpose: Used for monitoring short-lived jobs that do not run continuously.
     - The job runs, generates metrics, and pushes them to Prometheus via Pushgateway.
     - Alternatively, Prometheus scrapes the job’s metrics if it exposes an endpoint.
     - **Use Case:** Useful for CI/CD pipelines, ETL jobs, or database migrations.
3. **Scheduled Job** - Purpose: Monitors jobs that run at specific intervals (cron jobs, periodic tasks).
     - The job exposes metrics at an HTTP endpoint.
     - Prometheus scrapes these metrics at defined intervals.
     - **Use Cas**e: Scheduled backups, system cleanup tasks, or periodic health checks.
4. **Prometheus Configuration** (**prometheus.yml**) - the core configuration file for Prometheus
5. **Alertmanager - alertmanager.yml** - Handles alerts sent by Prometheus, manages deduplication, silencing, and routing.
     - Prometheus pushes alerts based on defined rules.
     - Alertmanager routes alerts to Email, Slack, PagerDuty, etc.
6. **Data Storage** (Local & Remote)
     - **Local Storage:**
          - Prometheus stores metrics on disk (/prometheus directory).
          - Limited retention (--storage.tsdb.retention.time=15d).
     - **Remote Storage:**
          - External solutions (Thanos, Cortex, InfluxDB, VictoriaMetrics) extend storage.
7. **Prometheus Server API** - Allows querying, managing, and retrieving Prometheus metrics via HTTP API
8. **PromQL** (Prometheus Query Language) - Used to query and analyze metric data
9. **Standalone & Self-Contained**
     - **Standalone Mode:**
          - Runs as a single binary (prometheus).
          - Stores data locally.
          - Best for small setups or testing.
     - **Self-Contained Mode:**
          - Bundles all components in a single deployment.
          - Common in on-premises monitoring.
10. **Metrics Storage & Retention**
     - **Metrics Storage:**
          - Stored in time-series format under /prometheus/data.
     - **Retention:**
          - Default: 15 days.
          - Configurable using: --storage.tsdb.retention.time=30d.
     - **Use Case**: Ensures efficient disk usage and supports historical analysis.
11. **Prometheus Operator**
     - Purpose: Simplifies deployment and management of Prometheus in Kubernetes.
     - **Key Features:**
          - Deploys Prometheus, Alertmanager, and related components.
          - Uses Custom Resource Definitions (CRDs) (Prometheus, Alertmanager, ServiceMonitor).
12. **kube-state-metrics**
     - Purpose: Collects detailed Kubernetes object metrics (Deployments, Pods, Nodes, etc.).
     - Runs inside the cluster, exposing metrics at /metrics.
13. **Prometheus Node Operator**
     - Purpose: Manages node-related monitoring in Kubernetes.
          - Deploys node-exporter to scrape system metrics.
     - Use Case: Automates node-level monitoring.
14. **Prometheus Alertmanager Operator**
     - Purpose: Manages Alertmanager instances in Kubernetes.
     - Uses CRDs (Alertmanager) to define alert routing.
     - Use Case: Automates alerting infrastructure in Kubernetes.

#### Ports Used in Prometheus and Grafana Setup

|Component	|Default Port	|Description|
|------|-----|------|
|Prometheus Server	|9090	|Web UI & API for querying and monitoring|
|Prometheus Node Exporter	|9100|	Exposes system metrics (CPU, memory, disk, network)|
|kube-state-metrics|	8080|	Exposes Kubernetes object metrics|
|Alertmanager	|9093	|Handles alerts from Prometheus|
|Pushgateway (optional)	|9091	|Accepts pushed metrics from short-lived jobs|
|Grafana Web UI	|3000	|Dashboard and visualization interface|

#### SPLUNK Architecture
- Splunk is a platform for searching, monitoring, and analyzing machine-generated big data through a web interface. It collects data from IT systems, especially logs, to gain insights into system performance, security, and operations.

**Key Components:**
- **Splunk Indexer:** Processes and stores data in an optimized format for fast searches.
- **Splunk Forwarder:**
     - **Universal Forwarder (UF)**: Lightweight agent that sends data to the indexer.
     - **Heavy Forwarder (HF)**: Resource-intensive forwarder that parses, indexes, and forwards data.
- **Splunk Search Head:** Interface for querying data, creating dashboards, and generating reports.
- **Splunk Deployment Server:** Manages configurations for Splunk instances.

#### Splunk (Enterprise Observability & Security)
- log management and observability platform used for monitoring, analyzing, and visualizing machine-generated data.
- It consists of multiple components that work together to collect, index, search, and analyze logs from various sources
- **Key Components**
     - Universal Forwarder
     - Heavy Forwarder
     - Indexer
     - Search Head
     - Deployment Server (DS) & Cluster Manager
     - Splunk Web & CLI
- **working process**
     - Forwarders (UF/HF) collect logs from applications, servers, Kubernetes clusters, and cloud platforms.
     - Indexers receive and store logs for efficient retrieval.
     - Search Heads allow SREs to analyze logs, debug issues, and create alerts.
     - Deployment Server ensures configurations are consistent across all Splunk agents.
     - Splunk Web/Dashboards provide real-time monitoring and insights
  
**How Splunk Works:**
- Data Collection: Collects logs, metrics, and machine data in real-time or from historical logs.
- Data Indexing: Processes and indexes data into structured events for easy searching.
- Search & Analysis: Users query indexed data with Splunk Processing Language (SPL).
- Visualization & Reporting: Dashboards, charts, and alerts help interpret data for actionable insights.
- Alerts & Monitoring: Configured to monitor thresholds and send real-time alerts for issue detection.

Use Cases:
- **Log Management:** Collect, analyze, and visualize logs from various systems.
- **SIEM:** Detect and investigate security incidents.
- **APM:** Analyze application performance and troubleshoot issues.
- IT Operations: Monitor infrastructure and detect operational issues.
- Business Intelligence: Gain insights into business metrics like customer activity.

Benefits:
- Real-Time Monitoring: Detect and alert on issues immediately.
- Scalability: Splunk can scale horizontally to meet the needs of both small and large enterprises.
- Flexibility: Supports various data sources and integrates with third-party tools.
- Search Power: SPL enables powerful searches across large datasets.

Splunk in DevOps:
- Log and Metrics Monitoring: Ingests logs from multiple services to monitor system health.
- Incident Detection & Resolution: Quickly identifies issues to reduce downtime.
- Automation: Integrates with DevOps tools for automated workflows based on detected patterns (e.g., triggering alerts in CI/CD pipelines).

#### Monitoring metrics
1. **System and Infrastructure Metrics**
     - CPU Usage: Utilization, load average, per-core usage
     - Memory Usage: Total, used, free, swap usage
     - Disk Usage: Read/write speeds, IOPS, disk space usage
     - Network Metrics: Bandwidth, packet loss, latency, errors
3. **Application Performance Metrics**
     - Response Times: Average, percentiles (P50, P90, P99)
     - Request Rates: Total requests, requests per second (RPS)
     - Error Rates: HTTP 4xx, 5xx errors
     - Latency: Database query time, API response times
4. **Database Metrics**
     - Query Performance: Slow queries, query execution time
     - Connections: Active connections, connection pool usage
     - Read/Write Operations: Query counts, insert/update/delete rates
5. **Container and Kubernetes Metrics**
     - Pod and Node Status: Running, pending, failed pods
     - Resource Utilization: CPU/memory usage per pod, node, or namespace
     - Cluster Health: Node availability, event logs, restart counts
6. **Logging and Observability Metrics**
     - Log Volume: Total logs, logs per second
     - Error Logs: Count of errors, warnings, or critical logs
     - Tracing Metrics: Distributed traces, span durations
7. **Business and Custom Metrics**
     - User Activity: Active users, session durations
     - Revenue and Transactions: Sales, conversions, payments
     - Custom Events: Any business-specific data visualization
8. **Security Metrics**
     - Authentication Failures: Failed login attempts
     - Intrusion Attempts: Unusual access patterns
     - Firewall Metrics: Blocked requests, allowed traffic
9. **IoT and Sensor Metrics**
     - Temperature, Humidity, Pressure: Sensor data visualization
     - Device Status: Online/offline state, error reports

#### Prometheus (Monitoring & Alerting)
- **Infrastructure Metrics:** CPU, memory, disk usage, network latency, etc.
- **Application Performance:** API response times, request rates, error rates.
- **Kubernetes Monitoring:** Pod status, resource utilization, node health.
- **Database Performance:** Query latency, connection pool usage.
- **Custom Business Metrics:** User activity, transactions per second, etc.

#### Setting up Prometheus & Grafana
1. **Setting Up Prometheus**
     - **prometheus.yml** (Prometheus Configuration)
     - Create a prometheus.yml file to define scrape jobs for monitoring Kubernetes nodes, services, and applications
2. **Setting Up Grafana**
     - Grafana Configuration File (grafana.ini)
     - Grafana requires a configuration file to define the database and security settings.
3. **Deploying on AWS/Azure**
     - Docker-Compose File
     - we can use docker-compose to deploy both Prometheus and Grafana.
4. **Deploying in Kubernetes (AWS EKS / Azure AKS)**
     - Prometheus Deployment (prometheus-deployment.yaml)
     - Prometheus ConfigMap (prometheus-configmap.yaml)
     - Grafana Deployment (grafana-deployment.yaml)
5. **Accessing Monitoring Dashboard**
     - Prometheus UI → **http://public-ip:9090**
     - Grafana UI → **http://public-ip:3000** ( login: admin/admin123 )
6. **Connecting Grafana to Prometheus**
     - Log in to Grafana (http://public-ip:3000).
     - Navigate to Configuration → Data Sources.
     - Add a new data source and select Prometheus.
     - Set http://prometheus:9090 as the data source URL.
     - Save & Test.
7. **Optional: Monitoring AWS/Azure Cloud Services**
     - Use AWS CloudWatch Exporter to monitor AWS services.
     - Use Azure Monitor Exporter for Azure services.
     - Set up Prometheus Alertmanager for alerts.

#### Prometheus & Grafana Architecture
- Data from Prometheus, InfluxDB, Elasticsearch, etc.
- Time-series analytics: Trends over time for system performance.
- Real-time dashboards: Monitoring business KPIs, server health, and app performance.
- Alerting & Notifications: Trigger alerts based on metric thresholds.

#### Dashboards 
- Creating dashboard involves several core components:
     - **Panels**:		# Panels are the individual visualizations within a Grafana dashboard, like graphs, tables, and gauges.
     - **Data Sources**:	# Grafana supports numerous data sources; you need to configure these to pull in relevant data.
     - **Variables**:	# Variables are dynamic filters that let you update data across the dashboard in real-time.
     - **Queries**:		#  Each panel uses a query to retrieve data from the selected data source, enabling customization of metrics displayed.
     - **Alerts**:		# Grafana allows setting alerts that notify users when data crosses specific thresholds, helping teams stay proactive.

1. **Grafana (For Prometheus, Datadog, CloudWatch, etc.)**
     - Open Grafana and navigate to Dashboards → Create → New Dashboard.
     - Click "Add a new panel" and select Prometheus (or other data source).
     - Choose Visualization Type (Graph, Gauge, Heatmap, etc.).
     - Click Save & Apply to add it to the dashboard.

2. **Dashboard in Amazon CloudWatch**
     - Open CloudWatch Console → Go to Dashboards.
     - we have to Create or Select existing dashboard.
     - we have to Add a Widget and select Graph, Number, or Text.
     - we have to Choose Metrics and select a service (e.g., EC2, Lambda, EKS).
     - we have to search filters to find the required metric.
     - we can Customize Widget like Setting time range, statistic type (Average, Sum ).
     - we need to Create Widget and then Save Dashboard.
       
3. **Kibana (For ELK - Elasticsearch Logs & Metrics)**
     - Open Kibana and go to Dashboard → Create New Dashboard.
     - Click "Add a visualization" and choose a chart type (e.g., Line, Bar, Pie).
     - Select Elasticsearch Index and configure filters (e.g., status:500).
     - Use KQL query for custom metrics, ( KQL Kibana Query Language)
     - Save and add it to the dashboard.

#### Grafana Dashboards creation
1. **Install Grafana**
     - There are several methods to install Grafana:
     - Using Docker: docker run -d — name=grafana -p 3000:3000 grafana/grafana
     - Using a Package Manager: For instance, brew install grafana on macOS.
     - Manual Download: You can download and install Grafana from the official website.
     - Once installed, Grafana can be accessed at http://localhost:3000 (the default port) and logged into with default credentials (admin/admin).
2. **Add a Data Source**
     - In Grafana, go to Configuration > Data Sources.
     - Choose the data source you need, such as Prometheus, MySQL, or Elasticsearch.
     - Enter the required connection details, like the URL for Prometheus or credentials for MySQL.
     - Click Save & Test to confirm the connection.
3. **Create a New Dashboard**
     - Click the + icon on the left-hand menu and select Dashboard.
     - Choose Add new panel to start creating your first panel.
     - Select a visualization type (such as graph, gauge, pie chart) based on your data requirements.
4. **Configure the Panel**
     - In the panel, select the data source.
     - Write a query to retrieve the desired data. For example, in Prometheus, a query might be rate(http_requests_total[5m]).
     - Customize the panel options to suit your needs:
          - Set the title, description, and display options.
          - Adjust the visualization style, including axes, colors, and legends.
5. **Set Variables (Optional)**
     - Variables allow you to create dynamic dashboards:
     - Go to Dashboard Settings > Variables > New.
     - Define the variable name and select the type (e.g., Query).
     - Create a query based on the data source. For instance, with Prometheus, a variable query for different instance values could include server IPs or application labels.
     - Save the variable. The dashboard will now have a dropdown to filter data based on this variable.
6. **Adding Alerts (Optional)**
     - Grafana allows users to set up alerts to proactively monitor their data:
     - Go to the Alert tab within the panel settings.
     - Define an alert condition (e.g., “CPU load average exceeds 80%”).
     - Set a time range and frequency for checking the alert.
     - Specify a Notification Channel (such as email, Slack, or PagerDuty).
7. **Save the Dashboard**
     - Click Save Dashboard in the top-right corner.
     - Name and save your dashboard, which can be shared with team members as needed.

#### ELK Stack (Elasticsearch, Logstash, Kibana) (Logging & Analysis)
- componentes
     - **Filebeat (optional)**: Collects and forwards logs to Logstash or Elasticsearch.
     - **Logstash:** Processes incoming logs and sends them to Elasticsearch.
     - **Elasticsearch:** Stores and indexes log data for fast querying.
     - **Kibana:** Provides a web interface to visualize log data and create dashboards.
- Logs
     - **Application Logs:** Error logs, access logs, debug logs.
     - **System Logs:** Syslogs, authentication logs, security logs.
     - **Network Traffic Logs:** Firewall logs, DNS queries, packet analysis.
     - **Business Intelligence:** Customer behavior, website analytics.
     - **Anomaly Detection:** Identifying security threats or system failures.

#### Datadog (Full-stack Monitoring & Observability)
- Datadog is a full-stack observability platform that help to monitor, troubleshoot, optimize applications and infrastructure.
     - Datadog Agent : lightweight process which rung on hosts to collect metrics, logs, and traces and sends data to the Datadog backend for analysis
     - Infrastructure Monitoring : Provides real-time monitoring of cloud, on-prem
     - Application Performance Monitoring (APM & Tracing)
     - Log Management
     - Synthetic Monitoring
     - Security Monitoring (CSPM, SIEM)
     - Real User Monitoring (RUM)

- **Infrastructure Monitoring:**		# CPU, memory, disk, networking, cloud services.
- **Application Performance Monitoring (APM):**	# Tracing distributed requests, service dependencies.
- **Log Management:**				# Aggregating, analyzing, and searching logs.
- **Security Monitoring:**			# Threat detection, vulnerability scanning.
- **Synthetic & Real User Monitoring (RUM):**	# User experience tracking for web applications.

#### Monitoring information
1. **Monitoring & Observability**
     - Infrastructure Monitoring – Monitors cloud and on-prem infrastructure, including servers, virtual machines, and cloud instances.
     - Network Monitoring – Provides visibility into network traffic, latency, and connectivity issues.
     - Container Monitoring – Tracks containerized environments (Docker, Kubernetes) and their performance.
     - Serverless – Monitors serverless functions like AWS Lambda, Azure Functions, and Google Cloud Functions.
     - Cloud Cost Management – Helps optimize cloud spending by tracking and analyzing usage.
     - Cloudcraft – A visualization tool for designing and monitoring cloud environments.
     - Application Performance Monitoring (APM) – Monitors application performance, traces transactions, and detects bottlenecks.
     - Service Catalog – Centralized view of microservices, dependencies, and ownership details.
     - Universal Service Monitoring – Provides automatic discovery and monitoring of services across environments.
     - Data Streams Monitoring – Tracks real-time data streams to ensure reliability and performance.
     - Data Jobs Monitoring – Observability for scheduled data jobs (ETL processes, batch jobs).
     - Database Monitoring – Monitors database performance, queries, and bottlenecks.
     - Continuous Profiler – Always-on CPU, memory, and resource profiling for applications.
     - Dynamic Instrumentation – Allows on-the-fly instrumentation without code changes.
2. **Logging & Security**
     - Log Management – Collects, indexes, and analyzes logs from different sources.
     - Sensitive Data Scanner – Detects and masks sensitive information in logs.
     - Audit Trail – Maintains a record of configuration changes and user actions.
     - Observability Pipelines – Routes observability data efficiently to control cost and performance.
     - Cloud Security Management – Identifies misconfigurations and security threats in cloud environments.
     - Cloud Security Posture Management (CSPM) – Ensures compliance and best practices in cloud security.
     - Cloud Workload Security – Protects cloud workloads by detecting runtime threats.
     - Identity & Entitlement Management – Monitors access control and permissions.
     - Application Security Management – Security monitoring for applications and APIs.
     - Software Composition Analysis (SCA) – Scans dependencies for vulnerabilities.
     - Code Security – Detects vulnerabilities in application code.
     - Runtime Code Analysis (IAST) – Identifies security flaws in applications at runtime.
     - Cloud SIEM (Security Information & Event Management) – Threat detection and security monitoring for cloud environments.
3. **Digital Experience Monitoring (DEM)**
     - Browser Real User Monitoring (RUM) – Captures real-time user interactions with web applications.
     - Mobile Real User Monitoring (RUM) – Monitors mobile app performance and user behavior.
     - Product Analytics – Provides insights into user engagement and behavior.
     - Session Replay – Records and replays user sessions to analyze issues.
     - Synthetic Monitoring – Simulates user interactions to test application performance.
     - Mobile App Testing – Automated testing for mobile applications.
     - Continuous Testing – Automates software testing in CI/CD pipelines.
     - Error Tracking – Detects, aggregates, and prioritizes errors.
4. **DevOps & CI/CD Visibility**
     - CI Visibility – Observability for CI/CD workflows, tracking build performance.
     - Test Optimization – Prioritizes test execution to speed up CI pipelines.
     - Service Level Objectives (SLOs) – Defines and tracks service performance targets.
     - Incident Response – Manages and resolves incidents efficiently.
     - Event Management – Centralized event tracking and correlation.
     - Case Management – Organizes and tracks security incidents.
5. **AI & Automation**
     - Bits AI – AI-powered insights and anomaly detection.
     - Metrics – Collects and analyzes performance metrics.
     - Watchdog – AI-powered alerts for anomalies and trends.
     - LLM Observability – Monitors large language models (LLMs) for performance and issues.
     - AI Integrations – Connects with AI-based applications for monitoring.
     - Workflow Automation – Automates remediation and operational tasks.
     - App Builder – Develops custom internal applications.
6. **Collaboration & Dashboards**
     - CoScreen – Remote collaboration and screen sharing for teams.
     - Teams – Communication and collaboration tools.
     - Dashboards – Custom dashboards for visualizing metrics and logs.
     - Notebooks – Interactive reports combining metrics, logs, and annotations.
     - Mobile App – Mobile access to Datadog dashboards and alerts.
7. **Fleet & Access Management**
     - Fleet Automation – Automates infrastructure management at scale.
     - Access Control – Manages user permissions and security policies.
     - OpenTelemetry – Supports OpenTelemetry for distributed tracing and observability.
8. **Alerts & Integrations**
     - Alerts – Configurable notifications for performance issues.
     - Integrations – Connects with third-party services (AWS, Azure, Kubernetes, Slack, etc.).
     - IDE Plugins – Extends monitoring capabilities into development environments.
     - API – Provides programmatic access to Datadog features.
     - Marketplace – Offers third-party integrations and plugins.

### Databases
- database management includes deployment, availability, performance optimization, backup, security, and observability of databases in production environments.
- Relational Databases (RDBMS)
     - MySQL, PostgreSQL, Microsoft SQL Server, Oracle DB, Amazon RDS, Azure SQL
- NoSQL Databases
     - MongoDB, Cassandra, DynamoDB, CosmosDB, Redis
- In-Memory Databases
     - Redis, Memcached
- Time-Series Databases
     - InfluxDB, TimescaleDB, Prometheus (for metrics)

|Database|Best For|
|-|-|
|MySQL|Web applications, e-commerce|
|PostgreSQL|Data analytics, financial transactions|
|SQL Server|Enterprise applications, Microsoft ecosystem|
|Oracle DB|Mission-critical enterprise apps|
|Amazon RDS|Managed SQL with cloud scalability|
|Azure SQL|AI-powered SQL with Azure integration|
|MongoDB|NoSQL document storage, flexible schemas|
|Cassandra|High availability, distributed data|
|DynamoDB|Serverless key-value storage|
|CosmosDB|Global-scale, multi-model NoSQL|
|Redis|In-memory caching|

#### MySQL
Type: Relational Database
- Use Cases:
     - Web applications (WordPress, Shopify, Joomla)
     - Small-to-medium scale enterprise applications
     - E-commerce platforms
- Example:
     - A web application needs a database for storing user profiles, transactions, and product details.
     - MySQL is used to maintain ACID-compliant transactions in an e-commerce checkout system.
- Pros:
     - Open-source, widely used.
     - Replication and clustering support.
- Cons:
     - Vertical scaling can be expensive.
     - Lacks advanced features like JSONB indexing (available in PostgreSQL).

#### PostgreSQL
Type: Relational Database (Object-Relational)
- Use Cases:
     - Large-scale data processing.
     - Applications requiring JSON storage and advanced indexing.
     - Analytics, financial transactions.
- Example:
     - A banking system storing customer transactions, ensuring ACID compliance with multi-version concurrency control (MVCC).
     - PostgreSQL’s JSONB support allows storing semi-structured data along with relational data.
- Pros:
     - Supports complex queries, full-text search, JSON indexing.
     - Highly scalable with partitioning and sharding.
- Cons:
     - Requires more memory and tuning than MySQL.

#### SQL Server
Type: Relational Database
- Use Cases:
     - Enterprise-level applications (ERP, CRM)
     - Data warehouses, business intelligence
     - Cloud-based applications on Azure
- Example:
     - A healthcare system storing patient records securely using Transparent Data Encryption (TDE).
     - A large retail company using SQL Server for sales forecasting.
- Pros:
     - Advanced security with Always Encrypted.
     - Optimized for Windows-based applications.
- Cons:
     - High licensing costs.
     - Windows-centric (Linux support is improving).

#### Oracle Database
Type: Relational Database
- Use Cases:
     - Enterprise-grade mission-critical systems (Banking, Airlines, Government).
     - Applications requiring high transaction throughput.
     - AI-driven analytics, big data processing.
- Example:
     - A stock trading platform that processes thousands of transactions per second.
     - Airline reservation systems that require high availability (RAC – Real Application Clusters).
- Pros:
     - Industry leader in performance, scalability, security.
     - Supports multi-tenancy and blockchain tables.
- Cons:
     - High cost and complex licensing.
     - Requires expert administration.

#### RDS (Relational Database Servce)
- its a managed database service that makes it easy to set up, operate, and scale relational databases in AWS.
- It automates tasks like backups, patching, scaling, so developers can focus on applications instead of database maintenance
- it is used deploy and manage popular database engines such as MySQL, PostgreSQL, and SQL Server, and leverage advanced features like automated backups and scaling.	
- Type: Managed SQL Service
- Use Cases:
     - Cloud-native SQL database for scalable web applications.
     - Multi-AZ deployments for high availability.
     - Auto-scaling for traffic-heavy applications.
- Example:
     - An e-commerce app using MySQL on Amazon RDS with read replicas to handle high traffic.
- Pros:
     - Fully managed: Automated backups, scaling.
     - Supports MySQL, PostgreSQL, SQL Server, and more.
- Cons:
     - Limited control over tuning configurations.
     - Expensive compared to self-managed databases.

#### Azure SQL
Type: Managed SQL Service on Azure
- Use Cases:
     - Microsoft-centric applications needing seamless integration with Azure services.
     - Cloud applications needing auto-scaling.
     - AI-powered database optimization.
- Example:
     - A finance company storing transactional data in Azure SQL and using Power BI for analytics.
- Pros:
     - Built-in AI-powered performance tuning.
     - High availability with geo-replication.
- Cons:
     - More expensive than on-premise SQL Server.

#### MongoDB
Type: Document-Oriented NoSQL
- Use Cases:
     - Content Management Systems (CMS)
     - Big data applications requiring flexible schemas.
     - IoT data storage.
- Example:
     - A social media platform storing posts, comments, and likes in JSON format.
- Pros:
     - Schema-less design allows flexible data storage.
     - Sharding and replication for scalability.
- Cons:
     - Not ACID-compliant by default (requires transactions in newer versions).

#### Apache Cassandra
Type: Column-Family Store NoSQL
- Use Cases:
     - Time-series data (IoT, logs, event tracking).
     - Large-scale distributed applications.
- Example:
     - Netflix and Facebook use Cassandra to store massive amounts of user-generated data.
- Pros:
     - Linear scalability with decentralized architecture.
     - High availability even in case of node failures.
- Cons:
     - Complex data modeling.

#### DynamoDB
Type: Key-Value NoSQL
- Use Cases:
     - Serverless applications requiring high-speed key-value lookups.
     - Gaming leaderboards, session storage.
- Example:
     - A real-time multiplayer game stores player scores and profiles in DynamoDB.
- Pros:
     - Fully managed, auto-scaling.
     - Supports event-driven applications (AWS Lambda integration).
- Cons:
     - Query flexibility is limited.

#### CosmosDB
Type: Multi-Model NoSQL (Key-Value, Document, Graph, Column-Family)
- Use Cases:
     - Global applications needing multi-region replication.
     - AI and IoT applications.
- Example:
     - A global retail company uses CosmosDB to store customer purchase history with <10ms latency.
- Pros:
     - Multi-model support (SQL, NoSQL, Gremlin for Graph DB).
     - Guaranteed SLAs for performance.
- Cons:
     - Expensive compared to other NoSQL solutions.

#### Redis
Type: In-Memory Key-Value Store
- Use Cases:
     - Caching frequently accessed data.
     - Session storage in web applications.
- Example:
     - A news website caches trending articles in Redis to improve load times.
- Pros:
     - Extremely fast read/write operations.
     - Supports Pub/Sub messaging, Leaderboards.
- Cons:
     - Data is stored in-memory, so RAM is a limitation.

#### SQL and NoSQL

|Feature|SQL (Relational) Databases|NoSQL (Non-Relational) Databases|
|-|-|-|
|Data Model|Structured (Tables, Rows, Columns)|Flexible (Key-Value, Document, Column-Family, Graph)|
|Schema|Predefined, Rigid Schema|Dynamic Schema (Schema-less)|
|Scalability|Vertical Scaling (Scale-Up)|Horizontal Scaling (Scale-Out)|
|Transactions|ACID (Atomicity, Consistency, Isolation, Durability) Compliant|BASE (Basically Available, Soft state, Eventually consistent)|
|Best Use Cases|Structured data, Financial transactions, Reporting|Big data, Real-time analytics, Distributed applications|
|Examples|MySQL, PostgreSQL, Microsoft SQL Server, Oracle, MariaDB|MongoDB, Cassandra, DynamoDB, Redis, CouchDB|
