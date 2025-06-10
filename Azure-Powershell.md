

#### Azure infrastructure Design
- key components :
     - Networking (VNet, Subnets, NSGs, Firewalls)
     - Compute (VMs, Azure Kubernetes Service, App Services)
     - Storage (Blob Storage, Azure Files, Managed Disks) 
     - Identity and Security (Azure AD, RBAC, Managed Identities)
     - Scalability & Availability (Load Balancers, Auto-Scaling, Availability Zones, Azure Front Door)
     - Monitoring & Governance (Azure Monitor, Log Analytics, Policy, Cost Management)

Log in to Azure
- az login

Set Variables
```
RESOURCE_GROUP="MyResourceGroup"
VM_NAME="UbuntuVM"
LOCATION="eastus"
IMAGE="UbuntuLTS"
VM_SIZE="Standard_B2s"
ADMIN_USER="azureuser"
SSH_KEY_PATH="$HOME/.ssh/id_rsa.pub"  # Replace with your SSH key path
```
####  Azure services majorly used
- **Compute & Container Services**
     - **Azure Virtual Machines (VMs)** - For self-managed Kubernetes clusters (kubeadm) or jump/bastion hosts, Host a self-managed Kubernetes control plane (if not using AKS)
     - **Azure Kubernetes Service (AKS)** - Fully managed Kubernetes with autoscaling, networking, and security integrations, Deploy microservices-based applications with Helm, Ingress, and Service Mesh
     - **Azure Container Registry (ACR)** - Private registry to store container images securely, push Docker images from CI/CD pipelines before deploying to AKS
- **DevOps & CI/CD**
     - **Azure DevOps (Pipelines, Repos, Artifacts)** - End-to-end CI/CD solution for AKS workloads, Build & deploy apps to AKS using YAML pipelines with Helm/Kustomize.
     - **GitHub Actions (with Azure integrations)** - GitHub-hosted runners and Azure service authentication, Build & deploy apps to AKS using YAML pipelines with Helm/Kustomize.
- **Security & Identity Management**
     - **Azure Active Directory (Azure AD)** - Secure authentication and role-based access
     - **Azure Key Vault** - Secure storage of secrets, certificates, and encryption keys
     - **Azure Policy & Defender for Cloud** - Enforce governance and security best practices
- **Networking & Traffic Management**
     - **Azure Application Gateway (AGIC)** - Layer 7 Ingress Controller with built-in Web Application Firewall (WAF) , Securely expose AKS workloads to the internet with SSL termination
     - **Azure Front Door** - Global traffic distribution, multi-region disaster recovery 
     - **Azure Private Link** - Secure connections to Azure services from AKS without public exposure
     - **Azure Load Balancer** - Distribute traffic to AKS nodes at Layer 4 (TCP/UDP). Load balance microservices exposed via NodePort.
- **Monitoring & Logging**
     - **Azure Monitor & Log Analytics** - Centralized logging, alerts, and dashboards for AKS , Monitor pod-level CPU/memory usage and trigger alerts if thresholds are breached
     - **Azure Application Insights** - Distributed tracing and application monitoring , Track request latency and failure rates for microservices in AKS
     - **Azure Sentinel (SIEM)** - Cloud-native security information & event management
- **Storage & Databases**
     - **Azure Files & Azure Blob Storage** - Persistent storage solutions, Store logs, backups, and application data
     - **Azure Managed Databases** (PostgreSQL, CosmosDB, SQL Server) - Fully managed, scalable database services

#### Azure Pipelines
Azure Pipelines is a cloud-based CI/CD service that automates the build, test, and deployment process. It supports multiple languages, frameworks, and deployment targets like Azure, Kubernetes, VMs, containers, and on-prem servers
- **key components**
     - Pipelines – The main CI/CD workflow.
     - Stages – Logical groupings of jobs (e.g., Build, Test, Deploy).
     - Jobs – A set of steps executed on an agent.
     - Steps – Individual tasks such as running a script or deploying an app.
     - Triggers – Define when a pipeline runs (e.g., push, PR, schedule).
     - Artifacts – Build outputs stored for deployments.
     - Agents & Pools – Machines running the pipeline tasks.
     - Environments – Used for approvals, gates, and tracking deployments.
- types of Azure Pipelines
     - Build Pipelines (CI) – Automates compiling code, running tests, and creating artifacts.
     - Release Pipelines (CD) – Automates deployments to different environments.
     - Multi-Stage Pipelines – Combines CI/CD into a single YAML pipeline.

#### CI/CD Pipeline Flow in Azure DevOps
- work flow
     - Code Commit – Developer pushes code to Azure Repos/GitHub.
     - Trigger CI Pipeline – The pipeline starts on commit or pull request.
     - Build & Test – The code is compiled, unit tests are run, and an artifact is generated.
     - Publish Artifact – The built application is stored in Azure Artifacts or a container registry.
     - Deploy to Staging – The application is deployed to a test/staging environment.
     - Approval Process – Manual or automatic approvals before production deployment.
     - Deploy to Production – The final release is deployed to users.

#### Agent in Azure Pipelines
- An agent is a machine that executes the pipeline tasks. There are two types:
     - Microsoft-hosted agents – Provided by Azure, includes Windows, Linux, macOS.
     - Self-hosted agents – Installed on-premises or in a cloud VM.

#### Triggers in Azure Pipelines
- Triggers define when a pipeline should run.
     - Continuous Integration (CI) : Runs when code is pushed to a branch
     - Pull Request (PR) Triggers : Runs on PR creation or updates
     - Scheduled Triggers : Runs at a specified time (e.g., nightly builds)
     - Manual Triggers : Requires user approval to start

#### VM creation though cli

- Create a Resource Group
     - az group create --name $RESOURCE_GROUP --location $LOCATION

- Deploy the VM
```
az vm create \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --image $IMAGE \
  --size $VM_SIZE \
  --admin-username $ADMIN_USER \
  --ssh-key-values $SSH_KEY_PATH \
  --public-ip-sku Standard
```

- Open Port 22 (SSH)
     - az vm open-port --port 22 --resource-group $RESOURCE_GROUP --name $VM_NAME

- Connect to Your VM
     - az vm show -d -g $RESOURCE_GROUP -n $VM_NAME --query publicIps -o tsv
     - ssh azureuser@<PUBLIC_IP>

#### VM from Azure Portal

- Sign in with your Azure account.

- Create a Virtual Machine
     - Search for "Virtual Machines" in the search bar at the top.
     - Click "Create" → "Azure Virtual Machine".

- Configure Basic Settings
     - Subscription - Select your Azure subscription
     - Resource Group - Create a new one or use an existing one
     - Virtual Machine Name - e.g., UbuntuVM
     - Region - Choose the nearest or preferred Azure region
     - Image - Select Ubuntu 20.04 LTS or latest version
     - Size - Choose Standard_B2s (for cost-effective performance)
     - Authentication Type - Choose SSH public key for security
     - Username - e.g., azureuser
     - SSH Public Key - Paste your SSH key (from ~/.ssh/id_rsa.pub)

- Configure Disks
     - OS Disk Type: Choose Standard SSD (recommended) or Premium SSD for better performance.
	 - If you need extra storage, click "Create and attach a new disk".
	 
- Configure Networking
	 - Virtual Network: Leave as default or create a new one.
	 - Subnet: Default is fine unless you have a custom setup.
	 - Public IP: Select "Create new" (for remote SSH access).
	 - NIC Network Security Group (NSG):
	 - Choose Basic.
	 - Select Allow SSH (Port 22) to enable remote access.
	 
- Management & Monitoring
     - Disable Boot diagnostics (optional, saves costs).
     - Enable Auto Shutdown if you want to save costs when the VM is not in use.

#### VM with PowerShell

- Connect-AzAccount
- Define Variables
     - $resourceGroup = "MyResourceGroup"
     - $location = "EastUS"
     - $vmName = "UbuntuVM"
     - $vmSize = "Standard_B2s"
     - $image = "UbuntuLTS"
     - $adminUser = "azureuser"
     - $sshKeyPath = "$HOME\.ssh\id_rsa.pub"  # Adjust path for your SSH key

- Create Resource Group
     - New-AzResourceGroup -Name $resourceGroup -Location $location

- Configure Networking	 
     - Create a Virtual Network (VNet)
	      - $vnet = New-AzVirtualNetwork -ResourceGroupName $resourceGroup -Location $location `
          - -Name "$vmName-VNet" -AddressPrefix "10.0.0.0/16"

     - Create a Subnet
	      - $subnet = Add-AzVirtualNetworkSubnetConfig -Name "$vmName-Subnet" `
	      -   -VirtualNetwork $vnet -AddressPrefix "10.0.0.0/24"
	      - $vnet | Set-AzVirtualNetwork

     - Create a Public IP
	      - $publicIp = New-AzPublicIpAddress -ResourceGroupName $resourceGroup `
          -  -Location $location -Name "$vmName-PublicIP" -AllocationMethod Static

     - Create a Network Security Group (NSG)
	      - $nsg = New-AzNetworkSecurityGroup -ResourceGroupName $resourceGroup -Location $location `
	      -  -Name "$vmName-NSG"

     - Allow SSH (Port 22)
	      - $nsgRule = New-AzNetworkSecurityRuleConfig -Name "AllowSSH" -Description "Allow SSH Access" `
	      -   -Access Allow -Protocol Tcp -Direction Inbound -Priority 100 `
  	      - -SourceAddressPrefix * -SourcePortRange * `
	      -   -DestinationAddressPrefix * -DestinationPortRange 22
	      - $nsg | Add-AzNetworkSecurityRuleConfig -NetworkSecurityRule $nsgRule | Set-AzNetworkSecurityGroup

     - Create a Network Interface (NIC)
	      - $nic = New-AzNetworkInterface -ResourceGroupName $resourceGroup -Location $location `
	      - -Name "$vmName-NIC" -SubnetId $subnet.Id -PublicIpAddressId $publicIp.Id `
	      - -NetworkSecurityGroupId $nsg.Id

- Create the Ubuntu VM
     - Define Credentials
	      - $sshKey = Get-Content -Path $sshKeyPath
	      - $cred = New-Object System.Management.Automation.PSCredential ($adminUser, (ConvertTo-SecureString "AzurePassword123!" -AsPlainText -Force))
     - Create VM Configuration
	      - $vmConfig = New-AzVMConfig -VMName $vmName -VMSize $vmSize |
	      - Set-AzVMOperatingSystem -Linux -ComputerName $vmName -Credential $cred -DisablePasswordAuthentication |
	      - Set-AzVMSourceImage -PublisherName "Canonical" -Offer "UbuntuServer" -Skus "20_04-lts" -Version "latest" |
	      - Add-AzVMNetworkInterface -Id $nic.Id |
	      - Set-AzVMBootDiagnostics -Enable

     - Add SSH Key
	      - Add-AzVMSSHKey -VM $vmConfig -PublicKey $sshKey -KeyData $sshKey
     Deploy VM
	      - New-AzVM -ResourceGroupName $resourceGroup -Location $location -VM $vmConfig

- Get the Public IP & Connect
     - Get-AzPublicIpAddress -ResourceGroupName $resourceGroup -Name "$vmName-PublicIP" | Select-Object IpAddress
	 - ssh azureuser@<PUBLIC_IP>


#### high-availability architecture
- set up a high-availability
     - Use Azure Load Balancer for distributing traffic across multiple VMs.
     - Deploy VMs in Availability Zones for redundancy.
     - Use Azure Traffic Manager to route traffic across different regions.
     - Leverage Azure Scale Sets for auto-scaling VMs based on demand.
     - Use Azure Backup and Site Recovery for disaster recovery.

#### Azure Blueprints 
- it allow organizations to define repeatable sets of Azure resources, policies, and security settings.
- They help enforce governance, ensure compliance, and simplify infrastructure deployment at scale

#### Azure DevOps CI/CD pipeline
- key components
     - Repositories – Azure Repos (Git) for source code management.
     - Build Pipeline (CI) – Uses Azure Pipelines to compile, test, and package code.
     - Artifact Management – Stores build outputs (Azure Artifacts, GitHub Packages).
     - Release Pipeline (CD) – Deploys applications using environments and approvals.
     - Infrastructure as Code (IaC) – Uses Terraform, Bicep, or ARM templates for infra deployment.
     - Security & Compliance – Scanning, code quality checks, and policy enforcement. 

#### Multi-stage CI/CD pipeline
- Stage 1: CI (Build & Test)
     - Checkout code from Azure Repos.
     - Run unit tests & security scans.
     - Package and publish artifacts.
- Stage 2: CD (Deploy to Dev/Test)
     - Deploy to a test environment using Azure App Service or Kubernetes.
     - Run integration tests.
- Stage 3: Deploy to Staging/Prod
     - Manual or automated approval gates.
     - Deploy using Blue-Green or Canary deployment strategies.

#### Security in a CI/CD pipeline
- ensure security
     - Use service connections with least privilege (RBAC).
     - Scan for vulnerabilities using Microsoft Defender for DevOps.
     - Implement Secrets Management (Azure Key Vault, GitHub Secrets).
     - Apply branch policies and code reviews.
     - Enable audit logs & alerts for pipeline activities.

#### Core principles of DevOps
- List
     - Collaboration – Development & operations teams work together.
     - Automation – Automate build, test, deployment, and monitoring.
     - Continuous Integration & Continuous Delivery (CI/CD) – Ensure fast and reliable releases.
     - Monitoring & Feedback – Use observability tools to detect and respond to issues.
     - Security & Compliance (DevSecOps) – Integrate security from the beginning

#### Infrastructure as Code (IaC)
- IaC is the practice of managing infrastructure through code rather than manual processes. It ensures consistency, repeatability, and automation.
     - Tools: Terraform, Bicep, ARM templates, Ansible
     - Use Cases: Provisioning VMs, configuring networking, enforcing security policies.

#### Azure API Management
- Azure API Management (APIM) is a service for managing, securing, and monitoring APIs. It helps with:
     - Authentication & Authorization (OAuth2, JWT, Azure AD).
     - Rate Limiting & Throttling to prevent abuse.
     - Caching to improve performance.
     - Transformations (e.g., JSON to XML).
     - Monitoring & Logging via Azure Monitor
- secure an API using Azure API Management
     - Use OAuth2 or API keys for authentication.
     - Enable IP filtering to restrict access.
     - Apply rate limits & quotas to prevent abuse.
     - Use policies (e.g., JWT validation, request/response transformation).
     - Enable WAF (Web Application Firewall) to protect against attacks.

#### Azure AD
- its Microsoft’s cloud-based identity and access management service. It provides authentication, authorization, and security for users, applications, and devices in an organization.
- Key Features & Benefits:
     - Single Sign-On (SSO) – its an authentication method that allows users to log in once and gain access to multiple applications without re-entering credentials
     - Multi-Factor Authentication (MFA) – Enhance security with extra authentication steps.
     - Conditional Access – Define access rules based on user/device/location.
     - Role-Based Access Control (RBAC) – Assign permissions based on roles.
     - Identity Protection – Detect suspicious activities & protect identities.
     - Hybrid Identity – Integrate with on-prem Active Directory (AD).
- Core Components
     - Users & Groups
          - Users – Individual identities that can be employees, partners, or guests.
          - Groups – Collections of users for managing permissions (Security Groups, Microsoft 365 Groups).
     - Applications & Enterprise Apps
          - App Registrations – Used for integrating custom applications with Azure AD.
          - Enterprise Applications – Pre-integrated apps (e.g., Salesforce, ServiceNow).
     - Authentication Methods
          - Username & Password – Basic authentication.
          - Multi-Factor Authentication (MFA) – Adds an extra security layer (SMS, Authenticator App).
          - Passwordless Authentication – FIDO2 keys, Windows Hello, Authenticator App.
     - Role-Based Access Control (RBAC)
          - Assign specific permissions to users or groups.
          - Global Administrator – Full control over Azure AD.
          - User Administrator – Manages user identities.
          - Application Administrator – Manages app registrations.
- When to Use Azure AD?
     - When using cloud-based applications like Microsoft 365, Azure, or SaaS apps.
     - When requiring modern authentication (OAuth, SAML, OpenID Connect).
     - For zero-trust security with MFA, Conditional Access, and identity protection.
  
#### Azure Service Fabric - Architecture
- its a distributed systems platform for building scalable, reliable, and high-performance microservices-based applications.
- It allows you to build, deploy, and manage microservices and container-based applications efficiently

- Azure Service Fabric is a distributed systems platform for building and managing scalable microservices-based applications.
     - Supports stateful and stateless microservices.
     - Provides automatic scaling, health monitoring, and rolling updates.
     - Can run in Azure, on-premises, or multi-cloud environments.
     - Uses Reliable Services & Reliable Actors for high availability.
- Best Practice: Use Service Fabric Explorer for monitoring cluster health.

#### Azure Service Fabric components
- Architecture
     - Cluster – A set of machines (VMs or bare metal) running Service Fabric runtime.
     - Nodes – Individual machines (VMs or physical servers) in a cluster.
     - Applications – Collections of microservices running in the cluster.
     - Microservices – Can be stateless (API, worker process) or stateful (database, cache).
     - Reliability & Failover – Built-in mechanisms to handle failures automatically.
- Key Benefits:
     - Supports Stateful and Stateless Microservices – Unlike Kubernetes, it has built-in state management.
     - Automatic Scaling & Self-Healing – Auto-restarts failed services, reschedules workloads.
     - Multiple Programming Models – Supports Reliable Services, Reliable Actors, Containers, and Guest Executables.
     - Works On-Prem and in the Cloud – Can run in Azure, AWS, or on-premises.
     - Built-in Monitoring & Security – Integrated with Azure Monitor, RBAC, and encryption.
- different programming models
     - Reliable Services – Stateful or stateless microservices.
     - Reliable Actors – Actor-based distributed computing.
     - Guest Executables – Running any application as a process.
     - Containers – Deploying containerized apps using Service Fabric.

#### microservices-based application using Fabric
- Answer:
- Step 1: Set up Service Fabric Cluster
     - Deploy Azure Service Fabric Cluster in Azure.
     - Use ARM templates, CLI, or Azure Portal.
- Step 2: Create Microservices
     - Develop microservices using .NET Core, Java, or Node.js.
     - Choose between Stateful or Stateless services.
- Step 3: Deploy Application Package
     - Package microservices in a Service Fabric Application.
     - Deploy via Azure DevOps CI/CD pipelines.
- Step 4: Monitor & Scale
     - Use Service Fabric Explorer for real-time monitoring.
     - Enable automatic scaling for high availability.
- Best Practice: Implement rolling upgrades for zero-downtime deployments

#### Monitor Fabric applications
- Answer:
     - Use Azure Monitor & Application Insights for telemetry data.
     - Check Service Fabric Explorer for real-time cluster health.
     - Set up alerts for node failures, high CPU, and memory usage.
     - Enable log aggregation using Log Analytics & Kusto Query Language (KQL).
- Best Practice: Implement self-healing microservices to automatically restart failing services.

#### SSO - Single Sign-On
- its an authentication method that allows users to log in once and gain access to multiple applications without re-entering credentials
     -  Improves User Experience – No need to remember multiple passwords.
     -  Enhances Security – Reduces the risk of password-based attacks.
     -  Centralized Authentication – Users authenticate through one identity provider (IdP). 
- SSO Works (General Flow)
     - User accesses an application (e.g., Microsoft 365, Salesforce).
     - Application redirects to the Identity Provider (IdP) (e.g., Azure AD, Okta).
     - User logs in once (Username + Password, MFA).
     - IdP issues a security token (SAML, OAuth, OIDC).
     - Application validates the token and grants access.
- Setting Up SSO in Azure AD
- Step 1: Register the Application in Azure AD
     - Go to Azure Portal → Azure Active Directory.
     - Select App Registrations → New Registration.
     - Enter App Name, select Supported account types, and specify Redirect URI. - Click Register.
- Step 2: Configure Authentication
     - Under Authentication, select SAML, OpenID Connect, or OAuth.
     - Provide the Reply URL (ACS URL) given by the service provider.
- Step 3: Assign Users & Groups
     - Go to Enterprise Applications → Select the application.
     - Under Users and Groups, assign required users or security groups.
- Step 4: Enable SSO
     - Under Single Sign-On, select SAML or OAuth-based authentication.
     - Upload the Federation Metadata XML if using SAML.
     - Provide Issuer URL, SAML Certificate, and Endpoint URLs.
- Step 5: Test SSO & Deploy
     - Click Test SSO and verify authentication.
     - Deploy to production and monitor login activity in Azure AD Sign-In Logs.

#### Azure Front Door ( AFD )
its a cloud-based, global load balancer and CDN that enhances the performance, security, and availability of web applications by intelligently routing traffic. It’s designed for high availability, DDoS protection, and application acceleration.
- **Key Features**
     - Global Load Balancing
          - Routes traffic across multiple regions for high availability.
          - Uses anycast routing to send users to the nearest available backend.
     - Application Acceleration
          - Uses caching & edge optimizations to reduce latency.
          - Supports HTTP/2 and TLS termination for faster connections.
     - Intelligent Traffic Routing
          - Supports Path-based and Route-based load balancing.
          - Uses Session Affinity to maintain user sessions.
          - Can automatically failover to healthy backends.
     - Security & DDoS Protection
          - Web Application Firewall (WAF) integration for threat protection.
          - DDoS Protection to mitigate volumetric attacks.
          - Bot protection and rate limiting for malicious traffic.
     - API Gateway Capabilities
          - Provides SSL offloading to reduce backend processing.
          - Can act as a reverse proxy for microservices.
     - Integration with Azure Services
          - Works with Azure App Services, Azure Kubernetes Service (AKS), API Management, Virtual Machines (VMs)
- **How Azure Front Door Works**
     - User makes a request (e.g., https://example.com).
     - Front Door receives the request at the closest edge location.
     - Traffic is routed to the fastest/healthy backend.
     - Response is cached (if applicable) for future requests.
- **Use Cases**
     - Multi-region Web Applications – Ensures low latency & high availability.
     - E-commerce Websites – Improves site performance & security.
     - API Acceleration – Optimizes API responses globally.
     - DDoS Protection & Security – Safeguards against cyber threats.
     - Microservices & Containers – Works with Kubernetes (AKS).
- **Set Up Azure Front Door** (Quick Guide)
     - Go to Azure Portal → Create a new Front Door.
     - Configure a Front Door Profile (Standard/Premium).
     - Add Backends (App Service, Storage, VMs, etc.).
     - Set Routing Rules (Path-based, latency-based, etc.).
     - Enable Caching & Security (WAF, TLS, etc.).
     - Deploy & Test – Use a test domain (yourapp.azurefd.net).

#### DDoS
Distributed Denial of Service- its a cyberattack where multiple compromised devices (botnets) flood a target system with excessive traffic, causing slowdowns, crashes, or service disruptions.

**Protection Strategies**
- Network-Level Protection
     - Firewalls & Intrusion Prevention Systems (IPS) – Blocks malicious traffic at the network level.
     - Rate Limiting & Connection Throttling – Limits the number of connections per IP.
     - Geofencing & IP Blacklisting – Blocks traffic from specific regions or known attackers.
- Cloud-Based DDoS Mitigation Services
     - AWS Shield – Protects AWS workloads (CloudFront, ALB, EC2).
     - Azure DDoS Protection – Secures Azure workloads with automatic mitigation.
     - Cloudflare DDoS Protection – Detects and mitigates large-scale attacks globally.
     - Google Cloud Armor – Provides L7 DDoS protection.
- Content Delivery Networks (CDNs)
     - CloudFront, Cloudflare, Fastly, Akamai – Distribute traffic across multiple locations to absorb attacks.
- Web Application Firewalls (WAF)
     - Blocks malicious HTTP/S traffic (OWASP rules, bot mitigation).
     - Works with AWS WAF, Azure WAF, Cloudflare WAF, etc.
- Anomaly Detection & AI-Based Mitigation
     - Uses machine learning to detect unusual traffic patterns.
     - Example: AWS Shield Advanced, Azure Sentinel, Cloudflare Bot Management.

- Protect Your Infrastructure from DDoS Attacks
     - Enable DDoS Protection Services – AWS Shield, Azure DDoS, Cloudflare, etc.
     - Use a CDN – Absorbs traffic and reduces direct exposure.
     - Implement WAF Rules – Blocks bot attacks at L7.
     - Rate Limiting & CAPTCHAs – Reduces automated request flooding.
     - Monitor & Alert – Set up CloudWatch, Azure Monitor, or SIEM tools for anomaly detection.
     - Traffic Scrubbing – Services like Akamai Kona Site Defender filter out attack traffic.

#### Load Balancer
- Load balancing ensuring high availability, fault tolerance, and optimal performance for applications.
- It distributes incoming network traffic across multiple backend servers, preventing any single server from being overwhelmed and ensuring application reliability.
     - **Application Load Balancer (ALB):** For HTTP/HTTPS traffic and content-based routing.
     - **Network Load Balancer (NLB):** For TCP or UDP traffic with low latency and high throughput.
     - **Classic Load Balancer:** For older applications but typically replaced by ALB or NLB.
- STEPS
     - **Configure Security Settings:** SSL certificate , security group
     - **Define Target Groups:**	ec2 , container IP

- **Load Balancer for a web application on Azure**
     - I configure an Application Gateway (AGW) to distribute traffic across backend instances running in VM Scale Sets (VMSS) or Azure Kubernetes Service (AKS).
     - I create a Backend Pool with my instances, configure path-based routing (e.g., /api → API, /web → UI), and enable SSL termination using Azure Key Vault certificates. Security is enhanced with Azure WAF to block malicious traffic.
     - For scalability, the AGW integrates with VMSS and AKS Autoscaler, ensuring high availability and 99.99% uptime.
 
### Application Gateway vs Azure Load Balancers vs Front door

|Feature|	Azure Application Gateway	|Azure Load Balancer|	Azure Front Door|
|-|-|-|-|
|Layer|	Layer 7 (HTTP/S)|	Layer 4 (TCP/UDP)|	Layer 7 (HTTP/S)|
|Traffic Routing|	URL-based & host-based	|IP-based|	Global routing|
|Web Application Firewall (WAF)	|✅ Yes|	❌ No	|✅ Yes|
|SSL Termination	|✅ Yes|	❌ No|	✅ Yes|
|Multi-Region Traffic Distribution	|❌ No	|❌ No	|✅ Yes|
|Best Use Case|	Secure app-level routing & WAF protection|Load balancing for internal services|	Global traffic routing|

### Azure DevOps components
-
Azure DevOps Services  --- A suite of cloud-based DevOps tools that support collaboration, CI/CD, and application lifecycle management.
1. **Azure Repos**                # A Git-based version control system for managing source code.
2. **Azure Pipelines**            # A CI/CD service that automates builds, testing, and deployments.
3. **Azure Boards**               # Agile project management for tracking work items, sprints, and releases.
4. **Azure Test Plans**           # A testing suite for manual and automated testing.
5. **Azure Artifacts**            # A package management service for storing and sharing dependencies.

- indetails
1. **Azure Repos**
     - Supports both Git and TFVC (Team Foundation Version Control).
     - Provides branching strategies, pull requests, and code review processes.
     - Integration with Azure Pipelines for automated builds.
2. **Azure Pipelines**
     - Supports CI/CD automation for faster delivery.
     - Compatible with multiple languages (Java, .NET, Python, Node.js, etc.).
     - Multi-cloud deployment (Azure, AWS, GCP, Kubernetes).
     - Uses YAML or classic UI-based pipelines.
3. **Azure Boards**
     - Helps teams manage work items, backlogs, and sprints.
     - Supports Kanban and Scrum methodologies.
     - Provides dashboards and reports for tracking progress.
4. **Azure Test Plans**
     - Allows manual, exploratory, and automated testing.
     - Supports integration with Selenium, JMeter, and other test frameworks.
     - Provides bug tracking and reporting.
5. **Azure Artifact**s
     - Manages Maven, npm, NuGet, and Python packages.
     - Provides artifact versioning and retention policies.
     - Ensures secure package sharing across teams.
- **Additional Feature**s
     - Service Connections – Securely integrate external tools and cloud services.
     - Agent Pools – Manage self-hosted and Microsoft-hosted build agents.
     - Security & Compliance – Implement RBAC (Role-Based Access Control) and audit logs
     - Integration with GitHub – Enables seamless CI/CD for GitHub repositories.

#### Azure DevOps key activities
1. **CI/CD Pipeline Implementation & Optimization**
     - Design, develop, and manage Azure Pipelines for continuous integration and deployment.
     - Implement multi-stage YAML pipelines for automated builds, testing, and releases.
     - Ensure blue-green and canary deployments for minimal downtime.
2. **Source Code Management & Version Control**
     - Manage repositories using Azure Repos (Git) or integrate with GitHub.
     - Implement branching strategies (GitFlow, trunk-based development).
     - Set up pull request policies and code reviews.
3. **Infrastructure as Code (IaC)**
     - Use Terraform, ARM Templates, or Bicep to provision cloud resources.
     - Maintain version-controlled infrastructure in Azure Repos.
     - Automate environment setup with Azure DevOps Pipelines.
4. **Security & Compliance**
     - Implement RBAC (Role-Based Access Control) for Azure DevOps services.
     - Use Azure Key Vault for secrets management.
     - Perform vulnerability scanning in CI/CD pipelines.
5. **Monitoring & Logging**
     - Set up Azure Monitor, Application Insights, and Log Analytics.
     - Implement Prometheus/Grafana or ELK for logs and metrics.
     - Create automated alerts for failures and performance issues.
6. **Performance & Cost Optimization**
     - Optimize CI/CD pipeline efficiency to reduce build times.
     - Use self-hosted agents to minimize costs.
     - Implement auto-scaling and fault tolerance in cloud environments.
7. **Containerization & Kubernetes**
     - Build and manage Docker images for applications.
     - Deploy and manage workloads on Azure Kubernetes Service (AKS).
     - Use Helm charts for application packaging and deployment.
8. **Release Management & Blue-Green Deployments**
     - Set up multi-environment deployments (Dev, QA, Prod).
     - Use feature flags and deployment rings for controlled releases.
     - Automate rollback strategies for failed deployments.
9. **Incident Management & RCA**
     - Respond to incidents, failures, and service degradation.
     - Perform Root Cause Analysis (RCA) and implement corrective actions.
     - Improve system reliability using SLOs, SLIs, and SLAs.
10. **Collaboration & Documentation**
     - Work closely with developers, testers, security teams, and cloud architects.
     - Maintain documentation for pipelines, infrastructure, and troubleshooting.
     - Conduct DevOps best practices training for teams.

#### Azure DevOps - implement CI/CD pipelines
- Azure DevOps provides Azure Pipelines to automate CI/CD workflows.
1. Continuous Integration (CI):
     - Code is pushed to Azure Repos, GitHub, or other repositories.
     - Build pipeline (yaml/classic) triggers automatically.
     - Build stages include compilation, unit testing, security scanning, and artifact creation.
     - Artifacts are published to Azure Artifacts, Docker Hub, or other registries.
2. Continuous Deployment (CD):
     - Releases are managed in Azure Release Pipelines or multi-stage YAML pipelines.
     - Deployment can target Azure Kubernetes Service (AKS), Azure App Services, VMs, Functions, etc.
     - Approvals, manual gates, and rollback strategies are incorporated.
     - Infrastructure as Code (IaC) via Terraform, Ansible, or ARM templates is used for environment provisioning.
     - Monitoring & Logging via Application Insights, Log Analytics, or Prometheus/Grafana.

#### DevOps pipelines secure Azure 
1. Identity & Access Management:
     - Use Azure AD authentication and RBAC (Role-Based Access Control).
     - Implement least privilege access for service connections, agents, and secrets.
2. Secrets Management:
     - Store secrets in Azure Key Vault and integrate with pipelines.
     - Avoid hardcoding credentials in YAML files.
3. Pipeline Security:
     - Enable pipeline approvals & checks for sensitive deployments.
     - Use branch policies & PR validation to enforce reviews before merging code.
4. Code & Artifact Security:
     - Enable Azure Defender for DevOps to scan repositories for vulnerabilities.
     - Use code signing and artifact integrity verification.
5. Network & Compliance:
     - Restrict pipelines to trusted agent pools.
     - Use Private Links & Service Endpoints to isolate deployments.

#### Azure DevOps - blue-green or canary deployments 
- Azure DevOps supports progressive delivery strategies like blue-green, canary, and rolling deployments:
1. Blue-Green Deployment:
     - Maintain two identical environments (Blue & Green).
     - Deploy new changes to Green while keeping Blue live.
     - Switch traffic using Azure Traffic Manager, Application Gateway, or Kubernetes Ingress.
2. Canary Deployment:
     - Gradually roll out changes to a small subset of users.
     - Use Azure Front Door, Traffic Manager, or Feature Flags to route traffic dynamically.
     - Monitor telemetry (App Insights, Log Analytics) to detect issues before full rollout.
3. Rolling Deployment:
     - Deploy changes incrementally across multiple instances to avoid downtime.
     - Works well with Azure Kubernetes Service (AKS) and VM Scale Sets.

#### Azure devops pipelines - monitor and troubleshoot
- To ensure reliability in CI/CD pipelines, use:
1. Built-in Azure DevOps Logs:
     - Check real-time logs via the Azure Pipelines UI.
     - Use "Enable system diagnostics" for detailed logs.
2. Monitoring Tools:
     - Azure Monitor & Log Analytics for tracking pipeline runs.
     - Application Insights for end-to-end tracing of deployed applications.
     - Grafana & Prometheus for custom monitoring dashboards.
3. Alerts & Notifications:
     - Use Azure DevOps Service Hooks to trigger alerts in Slack, Teams, or PagerDuty.
     - Set up custom Azure Monitor alerts based on pipeline failures.
4. Troubleshooting Tips:
     - Debug agent logs if self-hosted runners fail.
     - Check YAML syntax errors using az pipelines validate.
     - Use retry strategies for transient failures in cloud environments.

#### Azure Resource Manager (ARM)
- its a deployment and management service for Azure. It provides a unified way to manage, provision, and organize Azure resources through declarative templates, APIs, and built-in role-based access control (RBAC).
- ARM allows you to deploy, update, and delete resources as a single unit, known as a resource group, ensuring consistency and control over cloud infrastructure.
- **Unified Resource Management**
     - ARM provides a single control plane to manage all Azure resources such as VMs, storage, databases, networking, Kubernetes clusters, etc.
     - It supports automation via Azure PowerShell, Azure CLI, REST API, SDKs, and Azure Portal.
- **Resource Groups**
     - A resource group is a logical container for Azure resources that share the same lifecycle.
     - All resources in a group can be deployed, updated, and deleted together.
     - Example: Grouping an Azure VM, Virtual Network, and Storage Account in a single resource group.
- **Declarative Infrastructure as Code** (IaC)
     - ARM enables Infrastructure as Code using ARM Templates (JSON-based files) to define, deploy, and update resources in a repeatable and automated manner.
     - Alternative to Terraform & Bicep.
- **Role-Based Access Control (RBAC)**
     - ARM integrates RBAC to define who can perform what actions on resources.
     - Access is managed via Azure Active Directory (Azure AD).
- **Tagging & Policy Enforcement**
     - Tags help in organizing resources (e.g., Environment: Production, Owner: DevOps Team).
     - Azure Policy ensures compliance with security & governance rules (e.g., enforce encryption, region restrictions).
- **Dependency Management & Consistency**
     - ARM ensures dependencies are resolved before provisioning.
     - Deployments remain idempotent, meaning running the same ARM template multiple times does not create duplicates.

#### Secure security of secrets in Azure Pipelines
- Security
     - Azure DevOps Secret Variables – Store sensitive data in Pipeline Variables ($(MY_SECRET)).
     - Azure Key Vault – Securely fetch secrets at runtime.
     - Service Connections – Authenticate deployments securely.
- 
     - Enable Access Control (RBAC) – Restrict access to pipelines.
     - Secrets Management – Store credentials in Azure Key Vault.
     - Service Principals – Secure cloud authentication.
     - Enable Pipeline Policies – Require approvals before deployment.
     - Enable CI/CD Auditing – Log and monitor pipeline run
 
#### AWS Storage Services

|Service|Type|Use Case|Durability Availability|Access Protocols|
|-|-| -|-|-|
|Amazon S3|Object Storage|Data storage, backups, web hosting|99.99% durability|HTTP/HTTPS|
|Amazon EBS|Block Storage|EC2 instance storage|99.999% availability|Block-level|
|Amazon EFS|File Storage|Shared file storage for multiple instances|99.9% availability|NFS|
|AWS Storage Gateway|Hybrid Cloud Storage|Backup, disaster recovery|Depends on configuration|NFS|
|RDS Storage|Database Storage|Managed relational database storage|99.95% availability|SQL-based|

#### Azure Storage Services

|Storage Type|Best For|Protocol|
|-|-|-|
|Blob Storage|Unstructured data, backups, media files|HTTP(S)|
|File Storage|Shared file storage, application file shares|SMB, NFS|
|Disk Storage|VM storage, persistent disk|Block storage|
|Table Storage|NoSQL key-value data|REST API, SDK|
|Queue Storage|Asynchronous messaging|REST API, SDK|
|Data Lake Storage|Big data analytics|HDFS, REST API|
|NetApp Files|High-performance file storage|NFS, SMB|

#### Routing
its network traffic flows between resources, like within the cloud or to the internet or to on-premises networks.

- **Routing Components in Azure**

|Azure Component	|Description|
|- | - |
|Internet Gateway (Default)	|Automatically allows internet access for public IPs.|
|Network Security Group (NSG)	|Controls inbound/outbound traffic with firewall rules.|
|Azure Firewall	|Provides advanced security policies and threat protection.|
|Load Balancer	|Distributes traffic across backend servers.|
|Azure Virtual Network Peering	|Connects multiple VNets.|
|Azure VPN Gateway	|Routes traffic between on-prem and Azure.|
|Azure ExpressRoute	|Dedicated high-speed link between on-prem and Azure.|

- **AWS vs Azure Routing – Key Differences**

|Feature	|AWS 	|Azure |
| - |- | - |
|Default Routing	|Uses Route Tables in VPC	|Uses System Routes in VNet|
|Custom Routing	|Route Tables + Transit Gateway	|User-Defined Routes (UDR)|
|Inter-VPC/VNet Routing	|VPC Peering or Transit Gateway	|VNet Peering|
|Internet Access	|Internet Gateway (IGW)	|Built-in Internet Gateway|
|Private Internet Access	|NAT Gateway / NAT Instance	|Azure NAT Gateway|
|On-Prem Connectivity	|VPN Gateway / Direct Connect	|VPN Gateway / ExpressRoute|
|Firewall Protection	|AWS Security Groups, AWS Firewall	|NSGs, Azure Firewall|

#### All ports information

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

### RCA  main info - Root Cause Analysis
#### RCA - critical production outage - Memory leak
- there was **critical production outage** that impacted customer transactions. This issue was caused by a **memory leak** in a microservice running on Kubernetes (EKS), leading to pod crashes and increased response times.
- We identified the issue by analyzing Kubernetes logs, Prometheus metrics, and application logs. The root cause was improper memory allocation due to an unoptimized Java application.
- To resolve the issue,
     1. Increased memory limits and requests in the Kubernetes deployment.
     2. Implemented JVM garbage collection tuning to prevent excessive memory usage.
     3. Set up alerts in Prometheus and Grafana to monitor memory utilization trends.
- As a preventive measure, we conducted load testing and implemented auto-scaling policies to handle traffic spikes efficiently. This RCA helped us improve system stability and observability.
--------------------
#### RCA - during a high-traffic event - slow response
- there was critical production outage that affected application **during a high-traffic event**. Customers experienced **slow response** times and transaction failures.
- Root Cause Analysis (RCA):
     - After investigating AWS CloudWatch logs, Kubernetes (EKS) metrics, and Prometheus alerts, we found that CPU and memory usage spiked abnormally.
     - The root cause was a memory leak in a microservice, which led to excessive garbage collection and eventually caused the pods to crash.
     - This happened because the Java application had unbounded in-memory caching, leading to out-of-memory (OOM) errors.
- Resolution Steps:
     1. Increased Kubernetes memory limits and requests to avoid premature OOM kills.
     2. Optimized Java garbage collection (G1GC tuning) to reduce heap memory pressure.
     3. Implemented a circuit breaker pattern using Istio to gracefully degrade service instead of failing completely.
     4. Scaled out EKS worker nodes to handle load spikes.
- Preventive Actions:
     - Set up memory utilization alerts in Prometheus and Grafana to detect anomalies earlier.
     - Performed stress testing to simulate high-traffic scenarios.
     - Introduced an automatic pod restart mechanism to handle memory leaks proactively.
- As a result, we reduced downtime by 60%, improved system resilience, and prevented further customer-impacting incidents.
---------------------------------------------
#### RCA - Database Connection Failures in Production
- Incident Summary:
     - Users reported intermittent application failures.
     - Error logs showed “Too many connections” to the database.
- **Root Cause Analysis (RCA):**
- Investigation Steps:
     - Checked application logs (/var/log/app.log) → Found frequent database connection timeouts.
     - Monitored database connections (SHOW PROCESSLIST in MySQL) → Max connections limit exceeded.
     - Kubernetes readiness probes were failing due to slow queries.
- Root Cause:
     - The application was not closing database connections properly, leading to connection leaks.
     - Auto-scaling was not configured, leading to increased load on a limited set of database instances.
- Resolution Steps:
     - Implemented connection pooling in the application.
     - Increased max_connections in MySQL (from 200 to 500).
     - Configured AWS RDS Auto Scaling to handle increased load.
     - Set up Grafana alerts for high connection usage.
- Preventive Measures:
     - Introduced automated database connection cleanup.
     - Conducted load testing to ensure the fix worked.
------------------------
#### RCA - High Latency in API Gateway Requests
- Incident Summary:
     - API response times increased from 100ms to 2s+, impacting user experience.
     - AWS ALB logs showed a spike in 504 Gateway Timeout errors.
- **Root Cause Analysis (RCA):**
- Investigation Steps:
     - Checked API Gateway logs → Requests were reaching backend but took longer to respond.
     - Examined EC2 instance metrics → High CPU utilization (~95%).
     - Analyzed application logs → Found that the service was waiting for an external API (third-party dependency).
- Root Cause:
     - A third-party API dependency was slow, causing bottlenecks.
	 - Retry logic was misconfigured, leading to increased CPU load.
- Resolution Steps:
     - Implemented a timeout mechanism for API calls.
     - Introduced circuit breaker pattern (using Istio).
     - Optimized auto-scaling policies to add more instances dynamically.
- Preventive Measures:
     - Implemented caching to reduce API calls.
     - Added SLO-based alerting to catch high latency early.
------
#### RCA - Data Loss Due to S3 Bucket Misconfiguration
- **Problem:** A developer accidentally deletes important objects from an S3 bucket that had no versioning enabled.
- Solution:
     - **Restore from Backups:** If AWS Backup or replication is enabled, restore from the latest backup.
     - **Enable Versioning & MFA Delete:** Prevent accidental deletions in the future.
     - **Use S3 Object Lock:** Enable retention policies for critical objects.
     - **Configure IAM Policies:** Implement least privilege access and enable logging using AWS CloudTrail.

#### RCA - High Latency in RDS Database Queries
- **Problem:** An Amazon RDS database suddenly experiences high latency, affecting application performance.
- Solution:
     - **Check CloudWatch Metrics:** Look for high CPU, memory, or IOPS utilization.
     - **Enable Performance Insights:** Identify slow queries and optimize them using indexing or query refactoring.
     - **Increase Instance Size:** Upgrade to a larger instance type if the load is high.
     - **Use Read Replicas & Caching:** Implement RDS Read Replicas and Redis/Memcached for better performance.
     - **Review Connection Pooling:** Ensure efficient use of database connections.

#### RCA - Auto Scaling Group Fails to Launch New Instances
- **Problem:** An Auto Scaling Group (ASG) is set up to scale automatically, but it fails to launch new instances when demand increases.
- Solution:
     - **Check IAM Role Permissions:** Ensure ASG has permissions to launch EC2 instances.
     - **Verify Launch Template/Configuration:** Ensure it references the correct AMI, instance type, and security groups.
     - **Check Quotas & Limits:** AWS account limits may prevent new instances from being created.
     - **Check AZ Availability:** If a specific AZ is out of capacity, allow multi-AZ deployment.
     - **Inspect Health Checks:** If ELB health checks fail, adjust thresholds or instance warm-up times.
#### RCA - Lambda Function Execution Timeout
- **Problem:** A Lambda function fails due to timeout issues, causing incomplete execution.
- Solution:
     - **Increase Timeout Limit:** Default is 3 seconds; extend it as needed (max 15 minutes).
     - **Optimize Code Execution:** Reduce response times by optimizing loops, API calls, and database queries.
     - **Use Asynchronous Invocation:** If processing is long, switch to SQS or SNS for async execution.
     - **Enable Provisioned Concurrency:** Reduce cold start delays for latency-sensitive functions.
     - **Monitor with CloudWatch Logs:** Identify bottlenecks using AWS X-Ray and CloudWatch

#### Production issues
#### RCA - critical production outage
- there was **critical production outage** that impacted customer transactions. This issue was caused by a **memory leak** in a microservice running on Kubernetes (EKS), leading to pod crashes and increased response times.
- We identified the issue by analyzing Kubernetes logs, Prometheus metrics, and application logs. The root cause was improper memory allocation due to an unoptimized Java application.
- To resolve the issue,
     - Increased memory limits and requests in the Kubernetes deployment.
     - Implemented JVM garbage collection tuning to prevent excessive memory usage.
     - Set up alerts in Prometheus and Grafana to monitor memory utilization trends.
- As a preventive measure, we conducted load testing and implemented auto-scaling policies to handle traffic spikes efficiently. This RCA helped us improve system stability and observability.

#### RCA **high-traffic event** due to this application was **slow response**
- there was critical production outage that affected application **during a high-traffic event**. Customers experienced **slow response** times and transaction failures.
- Root Cause Analysis (RCA):
     - After investigating AWS CloudWatch logs, Kubernetes (EKS) metrics, and Prometheus alerts, we found that CPU and memory usage spiked abnormally.
     - The root cause was a memory leak in a microservice, which led to excessive garbage collection and eventually caused the pods to crash.
     - This happened because the Java application had unbounded in-memory caching, leading to out-of-memory (OOM) errors.
- Resolution Steps:
     1. Increased Kubernetes memory limits and requests to avoid premature OOM kills.
     2. Optimized Java garbage collection (G1GC tuning) to reduce heap memory pressure.
     3. Implemented a circuit breaker pattern using Istio to gracefully degrade service instead of failing completely.
     4. Scaled out EKS worker nodes to handle load spikes.
- Preventive Actions:
     - Set up memory utilization alerts in Prometheus and Grafana to detect anomalies earlier.
     - Performed stress testing to simulate high-traffic scenarios.
     - Introduced an automatic pod restart mechanism to handle memory leaks proactively.
- As a result, we reduced downtime by 60%, improved system resilience, and prevented further customer-impacting incidents.

#### RCA - Database Connection Failures in Production
Incident Summary:
     - Users reported intermittent application failures.
     - Error logs showed “Too many connections” to the database.
- Investigation Steps:
     - Checked application logs (/var/log/app.log) → Found frequent database connection timeouts.
     - Monitored database connections (SHOW PROCESSLIST in MySQL) → Max connections limit exceeded.
     - Kubernetes readiness probes were failing due to slow queries.
- Root Cause:
     - The application was not closing database connections properly, leading to connection leaks.
     - Auto-scaling was not configured, leading to increased load on a limited set of database instances.
- Resolution Steps:
     - Implemented connection pooling in the application.
     - Increased max_connections in MySQL (from 200 to 500).
     - Configured AWS RDS Auto Scaling to handle increased load.
     - Set up Grafana alerts for high connection usage.
- Preventive Measures:
     - Introduced automated database connection cleanup.
     - Conducted load testing to ensure the fix worked.

#### RCA - EC2 Instance Suddenly Stops Responding
- Pinging the instance shows no response, and SSH access fails.
- Solution:
     - **Check AWS Console & CloudWatch Logs:** Verify if the instance is running and check system logs.
     - **Verify Security Groups & NACLs:** Ensure inbound/outbound rules allow necessary traffic.
     - **Check CPU & Memory Utilization:** If high, try restarting the instance.
     - **Check EBS Volume Health:** If the root volume is corrupted, detach it, attach it to another instance, and repair it.
     - **Recreate the Instance:** If nothing works, create a new instance from a backup or AMI.

#### Security & Compliance
How do you implement security best practices in a cloud-native environment?
- Security in a distributed system must be proactive, automated, and enforced at multiple layers.
1. Identity & Access Management (IAM):
     - Implement least privilege access (RBAC, ABAC).
     - Use OIDC, OAuth2, and AWS IAM roles for secure authentication.
2.  Network Security:
     - Enforce VPC isolation, private networking, and service mesh (Istio, Linkerd).
     - Implement Web Application Firewalls (AWS WAF, Azure Front Door).
3. Data Security & Compliance:
     - Encrypt data at rest (AWS KMS, HashiCorp Vault) and in transit (TLS 1.2+).
     - Ensure compliance with SOC 2, HIPAA, GDPR using policy-as-code.
4. Automated Security Scanning:
     - Embed SAST, DAST, and dependency scanning in CI/CD.
     - Enable runtime security monitoring (Falco, Sysdig).

### SOP - Standard Operating Procedures
#### SOP - recurring application crashes
- There was a pod failures in production because our on-call team was struggling with troubleshooting **application crashes efficiently**.
- SOP: Kubernetes Pod Failure Troubleshooting Guide
- Steps included:
     - Checking pod health and status (kubectl get pods --all-namespaces)
     - Fetching logs for debugging (kubectl logs <pod-name> -n <namespace>)
     - Checking resource utilization (kubectl top pods)
     - Restarting a problematic pod (kubectl delete pod <pod-name>)
     - Rolling back a deployment if needed (kubectl rollout undo deployment/<deployment-name>)
     - Escalation matrix: When to escalate an issue and whom to contact.
- Impact of the SOP:
     - Reduced incident resolution time by 40% as new engineers could quickly follow steps.
     - Improved on-call effectiveness, as they now had clear guidance.
     - Reduced unnecessary escalations, allowing senior engineers to focus on complex tasks.
- This SOP was well-documented in Confluence and shared across teams via Slack and email, ensuring quick accessibility

#### SOP - Pod gets stuck in CrashLoopBackOff or Pending state.
- SOP Steps:
     - Check the pod status:			#      - **kubectl get pods -n <namespace>**
     - Describe the pod to get error details:	#      - **kubectl describe pod <pod-name> -n <namespace>**
     - Check logs for failure reasons:		#      - **kubectl logs <pod-name> -n <namespace>**
     - Restart the pod manually (if required):	#      - **kubectl delete pod <pod-name> -n <namespace>**
     - Check node status (if pod scheduling fails):	#      - **kubectl get nodes**
     - Escalate to cloud provider if there is an underlying infrastructure issue.
- Outcome: This SOP reduced pod recovery time from 30 minutes to 5 minutes.

#### SOP - Restoring a Failed Jenkins Job, A CI/CD pipeline fails frequently, blocking deployments
- SOP Steps:
     - Identify the failed job in Jenkins UI.
     - Check the logs under "Console Output".
     - Verify SCM connectivity:			#      - **git pull origin main**
     - Check disk space on the Jenkins server:	#      - **df -h**
     - Restart Jenkins service if necessary:	#      - **systemctl restart jenkins**
     - Rerun the pipeline and validate the fix.
- Outcome: This SOP helped junior engineers resolve issues independently.
 
#### Migration
1. **Cloud Migration Strategy**
2. **Observability & Performance Monitoring**
3. **validate that an application is production-ready after migration**
4. **security best practices during cloud migration**
5. **Post-Migration Reliability & Cost Optimization**

**Cloud Migration Strategy**
1. **Assess & Plan:**
     - Define migration goals (performance, cost, resilience).
     - Choose the right migration strategy: Rehost (Lift & Shift), Replatform, Refactor, or Rebuild.
     - Perform dependency mapping (via CloudEndure, StratoZone, or AWS Migration Hub).
2. **Document Migration Plans:**
     - Create Application Runbooks (network, storage, scaling, backup)
     - Define SLIs, SLOs, and SLAs for performance & reliability tracking.
     - Maintain pre-migration architecture vs. post-migration state diagrams.
3. **Automate Migration Process:**
     - Use Terraform, Ansible, CloudFormation for infrastructure automation.
     - Implement CI/CD pipelines for app redeployment in the cloud.

**Observability & Performance Monitoring**
1. **Instrument Applications for Tracing & Logging**:
     - Implement OpenTelemetry for distributed tracing (Jaeger, Zipkin).
     - Use structured logging (JSON logs, centralized ELK stack, or Datadog).
2. **Monitor Infrastructure & Workloads:**
     - Deploy Prometheus + Grafana dashboards for real-time metrics.
     - Use AWS CloudWatch / Azure Monitor for cloud-native insights.
     - Set up automated anomaly detection (AI-based insights in Splunk, Datadog APM).
3. **Define Performance Baselines:**
     - Benchmark app performance before & after migration.
     - Conduct load testing with Locust, K6, or JMeter.
     - Set up alerting (Prometheus AlertManager, PagerDuty, Slack notifications).

**validate that an application is production-ready after migration**
1. **Define Acceptance Criteria:**
     - Validate latency, throughput, scalability, failover mechanisms.
     - Run performance benchmarks comparing pre- vs. post-migration SLIs.
2. **Test Across Different Environments:**
     - Canary Deployments to reduce risk.
     - Run chaos engineering tests (Gremlin, LitmusChaos) to check fault tolerance.
     - Conduct failover & disaster recovery tests.
3. **Create Automated Validation Pipelines:**
     - Use Terraform Compliance or OPA to check infrastructure policies.
     - Implement API contract testing (Postman, Pact) for service compatibility.
     - Conduct real-user monitoring (RUM) with New Relic, Google Lighthouse for web apps.

**security best practices during cloud migration**
1. **IAM & Access Controls:**
     - Implement least privilege IAM policies (AWS IAM, Azure AD, GCP IAM).
     - Enforce multi-factor authentication (MFA) & SSO.
2. **Data Security & Encryption:**
     - Encrypt data at rest & transit (TLS 1.2+, AWS KMS, HashiCorp Vault).
     - Mask sensitive data using DLP tools & automated scans.
3. **Security Automation & Compliance Checks:**
     - Use AWS Config, Azure Policy, or HashiCorp Sentinel for compliance as code.
     - Automate CIS benchmarks & security audits with tools like Trivy, Snyk.

**Post-Migration Reliability & Cost Optimization**
1. **Implement Auto-Scaling & Right-Sizing:**
     - Use HPA/VPA for Kubernetes workloads.
     - Optimize resource allocation with AWS Compute Optimizer, Azure Advisor.
2. **FinOps & Cost Monitoring:**
     - Use AWS Cost Explorer, GCP Billing, Azure Cost Management for tracking.
     - Implement budget alerts & anomaly detection.
3. **Continuous Optimization & Reliability Engineering:**
     - Run chaos experiments regularly.
     - Establish SLOs & error budgets to balance feature velocity & reliability.

#### Troubleshooting Cloud Migration Issues
Scenario 1: **High Latency After Migration**
- Issue: Users report increased latency in the cloud environment.
     - Check networking configuration (VPC peering, security groups).
     - Verify DNS resolution (Route 53, Azure DNS).
     - Optimize database connections (RDS connection pooling, read replicas).
     - Use CDN (CloudFront, Azure CDN) for static assets.
- Fix: Enabled AWS Global Accelerator → Reduced response time by 40%.

Scenario 2: **Data Loss During Migration**
- Issue: Some database records are missing after migration.
     - Check data integrity before & after migration (checksum validation).
     - Enable CDC (Change Data Capture) for real-time sync.
     - Implement DB snapshots & rollbacks.
     - Use AWS DMS / Azure DMS for seamless migration.
- Fix: Re-ran migration with incremental sync → No data loss reported.

Scenario 3: **Cost Spikes After Migration**
- Issue: Cloud costs unexpectedly increased post-migration.
     - Identify unused resources (orphaned instances, unoptimized storage).
     - Right-size instances with AWS Compute Optimizer / Azure Advisor.
     - Move workloads to spot instances / reserved instances.
     - Enable budget alerts & anomaly detection.
- Fix: Moved 50% of workloads to Spot instances → Reduced cost by 35%.

Scenario 4: **Application Downtime After Cutover**
- Issue: Users report frequent timeouts after the final cutover.
     - Check load balancer health checks (ALB, NLB).
     - Verify DNS propagation delays.
     - Debug database connection timeouts.
     - Use auto-healing mechanisms (Kubernetes liveness probes, self-healing scripts).
- Fix: Adjusted LB health check thresholds → Fixed downtime issues.

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

#### Azure Site Recovery ( ASR )
Azure Site Recovery (ASR) is a disaster recovery (DR) and migration service that helps in replicating on-premises workloads to Azure or another site. It ensures business continuity by automatically failing over workloads in case of failure.
- How ASR helps in migration
     - Replicates VMs (Hyper-V, VMware) or physical servers to Azure.
     - Supports Application-Consistent Replication for databases and mission-critical applications.
     - Once replicated, workloads can be failed over permanently, completing migration.
- Best Practice: Perform a test failover before committing migration to ensure minimal downtime.

#### Migrate on-premises VMs to Azure using ASR
- Answer:
- Step 1: Prepare Azure for ASR
     - Set up an Azure Recovery Vault.
     - Configure target storage accounts and networking.
- Step 2: Prepare On-Premises Environment
     - Install ASR agent on source VMs.
     - Configure replication policy (RPO, RTO settings).
- Step 3: Enable Replication & Monitor
     - Replicate VMs, physical servers, or workloads to Azure.
     - Use Azure Monitor for replication health.
- Step 4: Perform Test Failover
     - Validate migration without affecting production.
- Step 5: Perform Failover & Cutover
     - Initiate planned/unplanned failover.
     - Once validated, commit failover to complete migration.
- Best Practice: Use Azure Migrate to assess on-premises workloads before using ASR

#### Azure Backup
- Answer:
     - Create a Recovery Services Vault in Azure.
     - Install Azure Backup Server (MABS) or Microsoft Azure Recovery Services (MARS) Agent on on-premises servers.
     - Configure Backup Policies (full, incremental, retention settings).
     - Schedule Backup Jobs and enable encryption for security.
     - Monitor backup jobs using Azure Monitor.
- Best Practice: Enable Soft Delete to protect against accidental deletion of backups.

#### Restore a deleted Azure VM using Azure Backup
- Answer:
     - Go to Recovery Services Vault → Select Backup Items.
     -  Choose the VM backup and click Restore VM.
     -  Select the restore point (full VM restore or file-level recovery).
     -  Specify target VM settings and restore.
     -  Verify the restored VM and reconfigure networking & public IP if needed.
- Best Practice: Use Azure Backup Instant Restore for faster recovery.

#### Migrating monolithic applications to microservices
- Answer:
- Step 1: Identify Bounded Contexts
     - Break monolithic applications into independent services.
- Step 2: Choose the Right Compute Service
     - Use Azure Service Fabric for stateful services.
     - Use AKS (Kubernetes) for stateless containers.
- Step 3: Implement Service Communication
     - Use Azure Service Bus or Event Grid for messaging.
     - Implement API Gateway (Azure API Management) for request routing.
- Step 4: Ensure Security & Observability
     - Use Managed Identities & RBAC for secure access.
     - Implement logging & distributed tracing (App Insights, Azure Monitor).
- Best Practice: Follow the 12-Factor App Principles for cloud-native microservices

#### Virtual Machine connectivity issues
To troubleshoot Azure VM connectivity issues, follow these steps:
- Check VM Status:
     - Ensure the VM is running and not in a stopped/deallocated state.
     - Check the Azure Service Health dashboard for ongoing Azure outages.
-  Check Network Security Group (NSG) Rules:
     - Validate inbound/outbound NSG rules to ensure the required ports (e.g., 22 for SSH, 3389 for RDP) are allowed.
     - az network nsg rule list --resource-group <rg-name> --nsg-name <nsg-name>
- Check Azure Firewall or Route Table Issues:
     - Ensure User-Defined Routes (UDR) do not block outbound traffic.
     - If a VPN or ExpressRoute is used, verify its connection status.
- Check VM Boot Diagnostics:
     - Use Serial Console for Linux and Boot Diagnostics for Windows to view boot errors.
     - Run a ping test from within the VM to confirm network connectivity.
- Check Azure Load Balancer (if applicable):
     - Ensure the backend VM is healthy under Load Balancer probes.
- Best Practice: Use Azure Network Watcher to diagnose and monitor connectivity.

#### Resize VM
- To resize an Azure VM:
     - Check the current SKU compatibility:
          - az vm list-sizes --location <region>
     - Stop the VM (if needed for resizing to a different series)
          - az vm deallocate --resource-group <rg-name> --name <vm-name>
     - Resize the VM
          - az vm resize --resource-group <rg-name> --name <vm-name> --size Standard_D4s_v3
     - Restart the VM
          - az vm start --resource-group <rg-name> --name <vm-name>
- Best Practice: Use Autoscaling and Virtual Machine Scale Sets (VMSS) for dynamic workload management

###

#### PowerShell 
PowerShell is a task automation and configuration management framework developed by Microsoft. 
- It includes a command-line shell, a scripting language, and a configuration management framework that is built on the .NET framework.
- PowerShell enables IT administrators and DevOps professionals to automate administrative tasks by executing commands (cmdlets) and writing scripts.
- PowerShell can interact with various system components, including the Windows operating system, Active Directory, Azure, and other Microsoft services. 
- It is widely used for automation, configuration management, and infrastructure as code (IaC) operations.

- PowerShell Features 
- PowerShell offers several powerful features that make it an essential tool for IT automation and administration:
     - Object-Oriented: Unlike traditional command-line interfaces, PowerShell works with objects instead of plain text, making data manipulation easier.
     - Cmdlets: PowerShell provides built-in commands (cmdlets) for performing system administration tasks.
     - Scripting and Automation: Supports scripting capabilities for automating repetitive tasks.
     - Remote Management: Allows administrators to execute commands on remote systems using PowerShell Remoting.
     - Pipeline Functionality: Enables chaining of commands to process data sequentially.
     - Extensibility: Allows users to create custom cmdlets and functions.
     - Integration with .NET Framework: Provides access to .NET objects and libraries.
     - Security Features: Supports execution policies, script signing, and role-based access control (RBAC).
     - Task Scheduling: Automates scheduled jobs and background tasks.

#### Concepts
     - Variable ( Input , Automatic )
     - cmdlets
     - loops
     - Operators ( Comparison )
     - Format commands
     - Get-Commands
     - Copy
     - Brackets
     - Array

#### Real time use cases
- Automate Azure Virtual Machine Deployment
- Auto-Scale Azure Virtual Machines Based on CPU Usage
- Create and Assign RBAC Roles to Users
- Retrieve Azure Activity Logs for Security Auditing
- Automate Azure Storage Account Backup
- Enforce PowerShell Security by Setting Execution Policies
- Deploy an Azure Kubernetes Cluster Using PowerShell
- We can integrate PowerShell with Azure DevOps Pipelines
- We can retrieve logs from Azure using PowerShell
- We can manage RBAC permissions using PowerShell
- We can schedule PowerShell scripts using Task Scheduler
- We can automate Azure VM scaling using PowerShell
- we can handle errors in PowerShell using Try-Catch-Finally
- We can securely store and retrieve secrets in PowerShell
     - Azure Key Vault
     - Secure String
- We can manage Azure resources using PowerShell

#### Powershell Commands

#### Powershell Basic Commands

```
Get-Help - Displays help information about PowerShell commands.
     - Example: Get-Help Get-Service
Get-Command - Lists all available PowerShell commands.
     - Example: Get-Command -Noun Process
Get-Alias - Displays aliases for PowerShell cmdlets.
     - Example: Get-Alias ls
Set-ExecutionPolicy - Changes the execution policy for running scripts.
     - Example: Set-ExecutionPolicy RemoteSigned
Get-History - Displays the command execution history.
     - Example: Get-History
Clear-Host - Clears the screen (similar to cls in cmd).
     - Example: Clear-Host
```
 
#### Powershell File and Directory Management

```
Get-Location - Shows the current directory.
     - Example: Get-Location
Set-Location (cd) - Changes the current directory.
     - Example: Set-Location C:\Users
Get-ChildItem (ls, dir) - Lists the files and directories.
     - Example: Get-ChildItem -Path C:\Users
New-Item - Creates a new file or folder.
     - Example: New-Item -ItemType File -Path C:\temp\file.txt
Remove-Item - Deletes files or folders.
     - Example: Remove-Item C:\temp\file.txt
Copy-Item (cp) - Copies files or directories.
     - Example: Copy-Item C:\file.txt -Destination C:\backup\
Move-Item (mv) - Moves or renames files.
     - Example: Move-Item C:\file.txt -Destination C:\newfile.txt
```
 
#### Powershell Process Management

```
Get-Process (ps) - Lists running processes.
     - Example: Get-Process | Sort-Object CPU -Descending
Stop-Process (kill) - Stops a running process.
     - Example: Stop-Process -Name notepad
Start-Process - Starts a new process.
     - Example: Start-Process notepad.exe
```

#### Powershell Service Management

```
Get-Service - Lists all services.
     - Example: Get-Service | Where-Object { $_.Status -eq "Running" }
Start-Service - Starts a service.
     - Example: Start-Service -Name Spooler
Stop-Service - Stops a service.
     - Example: Stop-Service -Name Spooler
Restart-Service - Restarts a service.
     - Example: Restart-Service -Name Spooler
```

#### Powershell User and System Information

```
Get-Date - Displays the current date and time.
     - Example: Get-Date -Format "yyyy-MM-dd HH:mm:ss"
Get-EventLog - Retrieves event logs.
     - Example: Get-EventLog -LogName System -Newest 10
Get-ComputerInfo - Displays system information.
     - Example: Get-ComputerInfo | Select-Object CsName, OSName, WindowsVersion
Get-WmiObject - Fetches system details using WMI.
     - Example: Get-WmiObject Win32_OperatingSystem
```

#### Powershell Networking Commands

```
Test-Connection - Pings a remote system.
     - Example: Test-Connection google.com -Count 4
Get-NetIPAddress - Displays IP addresses.
     - Example: Get-NetIPAddress
Get-NetAdapter - Shows network adapter information.
     - Example: Get-NetAdapter | Select-Object Name, Status, MacAddress
Resolve-DnsName - Resolves a domain name to an IP address.
     - Example: Resolve-DnsName google.com
```

#### Powershell Text and Data Handling

```
Get-Content (cat) - Reads a file’s content.
     - Example: Get-Content C:\temp\log.txt
Set-Content - Writes data to a file.
     - Example: Set-Content C:\temp\log.txt -Value "New Entry"
Add-Content (echo >>) - Appends data to a file.
     - Example: Add-Content C:\temp\log.txt -Value "Additional Info"
ConvertTo-Json - Converts objects to JSON format.
     - Example: Get-Process | ConvertTo-Json
ConvertFrom-Json - Converts JSON data to PowerShell objects.
     - Example: $json | ConvertFrom-Json
```

#### Powershell Security and Permissions

```
Get-ACL - Retrieves access control list (ACL) for files/folders.
     - Example: Get-ACL C:\temp\file.txt
Set-ACL - Modifies ACL settings.
     - Example: Set-ACL -Path C:\temp\file.txt -AclObject (Get-ACL -Path C:\temp\file.txt)
Get-LocalUser - Lists local user accounts.
     - Example: Get-LocalUser
Get-LocalGroup - Lists local groups.
     - Example: Get-LocalGroup
Add-LocalGroupMember - Adds a user to a local group.
     - Example: Add-LocalGroupMember -Group "Administrators" -Member "JohnDoe"
```

#### Powershell Automation and Scripting

```
Invoke-Command - Runs commands remotely.
     - Example: Invoke-Command -ComputerName Server1 -ScriptBlock { Get-Service }
Start-Job - Runs a command in the background.
     - Example: Start-Job -ScriptBlock { Get-Process }
Wait-Job - Waits for a background job to complete.
     - Example: Wait-Job -Id 1
Receive-Job - Gets the output of a background job.
     - Example: Receive-Job -Id 1
```

#### Pipeline
A pipeline (|) is used to pass the output of one command (cmdlet) as input to another cmdlet. This allows for efficient data processing without needing intermediate variables.

```
Get-Process | Sort-Object CPU | Select-Object -First 5
```
- Get-Process retrieves a list of running processes.
- Sort-Object CPU sorts them by CPU usage.
- Select-Object -First 5 selects the top 5 processes consuming the most CPU.

- This chaining mechanism makes PowerShell efficient for handling structured data.

- **ByValue**: Passes objects directly
     - **Get-Process | Stop-Process**
- **ByPropertyName**: Matches property names between commands
     - **Get-Process | Select-Object -Property Name**

#### Execution Policy
- PowerShell's execution policy determines the level of permission for running scripts. This is a security feature designed to prevent unauthorized or malicious scripts from executing.

- Execution Policy Levels:
     - **Restricted** – Default setting; allows only interactive commands (no scripts).
     - **AllSigned** – Only runs scripts signed by a trusted publisher.
     - **RemoteSigned** – Runs scripts created locally but requires signatures for downloaded scripts.
     - **Unrestricted** – Runs all scripts without restriction but warns for downloaded ones.
     - **Bypass** – No restrictions; used in automation.
- Example: Check and Set Execution Policy

```Get-ExecutionPolicy   # Check current execution policy
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser  # Set new policy
```

#### Try, Catch, and Finally
Try, Catch, and Finally blocks in PowerShell are used for error handling in scripts.

- **Try**: Contains the main code block.
- **Catch**: Captures and handles errors.
- **Finally**: Executes regardless of success or failure.

Syntax and Example:

```Try {
    $result = 10 / 0   # This will cause an error (division by zero)
} Catch {
    Write-Host "An error occurred: $_"
} Finally {
    Write-Host "Execution completed."
}
```

#### CIM and WMI
|Feature	|CIM (Common Information Model)	|WMI (Windows Management Instrumentation)|
|-|-|-|
|Protocol	|Uses WS-Man (Web Services for Management)	|Uses DCOM (Distributed Component Object Model)|
|Performance|	Faster and optimized for remote management|	Slower due to DCOM overhead|
|Platform Support|	Works on Windows and cross-platform	|Primarily Windows-based|
|Cmdlets	|Uses Get-CimInstance	|Uses Get-WmiObject|
|Security	|More secure with WS-Man|	Uses older authentication methods|

```# CIM example
Get-CimInstance -ClassName Win32_OperatingSystem 

# WMI example
Get-WmiObject -Class Win32_OperatingSystem
```

#### Powershell Variable
A variable in PowerShell stores values and is denoted with a $ sign.

```$Name = "PowerShell"
Write-Host "Welcome to $Name"
```
- This outputs: **Welcome to PowerShell**

- Declare and create a variable
```
$greeting = "Hello, PowerShell!"
Write-Host $greeting
```

- rename a variable
     - PowerShell does not support direct renaming of variables. Instead, reassign the value:
```
$oldVar = "Data"
$newVar = $oldVar
Remove-Variable oldVar
```

#### Powershell Input variable
$input is an automatic variable in PowerShell that represents the input passed to a function or script block. It is primarily used inside pipelines.

```function Test-Input {
    process {
        "Processing: $_"
    }
}
1,2,3 | Test-Input
```

Here, $input receives values from the pipeline.

#### Powershell Different Types of Variables
- Detail
     - **Scalar Variables**: Store single values ($x = 10).
     - **Array Variables**: Store multiple values ($arr = @(1,2,3)).
     - **Hash Tables**: Store key-value pairs ($h = @{"A"=1;"B"=2}).
     - **Automatic Variables**: Pre-defined system variables ($PSVersionTable).

#### Powershell Array
- collection of values stored in a single variable

```
$fruits = @("Apple", "Banana", "Cherry")
Write-Output $fruits[1]  # Output: Banana

example :
$fruits += "Orange"
```

#### Powershell Automatic Variable
Automatic variables are pre-defined by PowerShell and store system-related information.
- Common Automatic Variables:
     - **$PSVersionTable** – Shows PowerShell version details.
     - **$?** – Indicates the success ($true) or failure ($false) of the last command.
     - **$_** – Represents the current object in the pipeline.
     - **$Error** – Stores error details.

#### Powershell Hash table
A hash table is a collection of key-value pairs, similar to a dictionary.

```$employee = @{ "Name"="John"; "Age"=30; "Role"="Admin" }
Write-Host $employee["Name"]
```

#### Powershell Execute script
- Steps:
     - Save the script as .ps1 file (e.g., script.ps1).
     - Open PowerShell and navigate to the script directory.
     - Run the script
          - **.\script.ps1**
     - If execution is restricted, enable it
          - Set-ExecutionPolicy RemoteSigned -Scope CurrentUser

#### Powershell Extending
- **Modules**: Collections of scripts and functions
     - Import-Module Azure
- **Snap-ins**: Legacy method for extending functionality.


#### Powershell cmdlets
Cmdlets are lightweight commands used in PowerShell.

```Get-Process   # Retrieves running processes
Get-Service   # Lists all services
Start-Service -Name "Spooler"   # Starts a service
```

Cmdlets follow the Verb-Noun naming convention

#### Powershell Loop
A loop in PowerShell allows repeated execution of a block of code.

```for ($i=1; $i -le 5; $i++) {
    Write-Host "Number: $i"
}
```
- Loops - PowerShell supports several looping constructs:
- **For Loop**
```
for ($i=1; $i -le 5; $i++) { Write-Host "Iteration $i" }
```
- **While Loop**
```
$count = 1
while ($count -le 5) { Write-Host "Count: $count"; $count++ }
```
- **Do-While Loop**
```
do { Write-Host "Executing" } while ($false)
```
- **Foreach Loop**
```
$list = @(1,2,3)
foreach ($item in $list) { Write-Host "Item: $item" }
```

#### Powershell Operators
- Types
     - Arithmetic Operators: + - * / %
     - Comparison Operators: -eq, -ne, -gt, -lt, -ge, -le
     - Logical Operators: -and, -or, -not
     - Assignment Operators: =, +=, -=
     - String Operators: -match, -replace
     - Array Operators: -contains, -in
     - Redirection Operators: >, >>

#### Comparison operator
- Comparison operators compare values
     - -eq	Equal to
     - -ne	Not equal to
     - -gt	Greater than
     - -lt	Less than
     - -ge	Greater than or equal to
     - -le	Less than or equal to

```
if (5 -gt 3) { Write-Host "5 is greater than 3" }
```

#### Format commands
- Format the data
     - **Format-Table** - Displays data in a tabular format.
     - **Format-List** - Displays data in a list format.
     - **Format-Wide** - Displays data in a wide format with limited details.
     - **Format-Custom** - Allows custom formatting of output.
```
     **Get-Process | Format-Table -AutoSize**
```

#### Copy 
- The Copy-Item cmdlet is used to copy files, registry keys, and folders in PowerShell.

```
#Syntax
Copy-Item -Path <Source> -Destination <Target> [-Recurse] [-Force]
#Examples
Copy-Item -Path "C:\source\file.txt" -Destination "C:\backup\file.txt"
Copy-Item -Path "C:\sourceFolder" -Destination "C:\backupFolder" -Recurse
Copy-Item -Path "HKLM:\SOFTWARE\MyKey" -Destination "HKLM:\SOFTWARE\BackupKey" -Recurse
```

#### Brackets
- types of brackets:
     - **{} (Curly Braces)**	- Used for script blocks, functions, and conditional statements.
     - **() (Parentheses)**  	- Used for method calls, expressions, and precedence.
     - **[] (Square Brackets)** 	- Used for array indexing and type declaration.

```
$numbers = @(1, 2, 3)  # Square brackets define an array
Write-Output ($numbers[0])  # Parentheses group expressions
```

#### Get-Command
Get-Command retrieves available commands
```
Get-Command -Noun Process   # Lists all commands related to "Process"
Get-Command -Verb Get       # Lists all commands starting with "Get"
```

#### Launch PowerShell ( Windows )
- Ways to open PowerShell:
     - From the Start Menu: Search for "PowerShell" and open it.
     - From Run Dialog: Press Win + R, type powershell, and press Enter.
     - From CMD: Open Command Prompt and type
          - powershell
     - With Administrator Rights: Right-click on "PowerShell" and select "Run as Administrator". 

#### Powershell VM Deployment
This script provisions an Azure VM with networking and a storage account.
- Key Features:
     - Creates a VM, VNet, Subnet, NSG, and Public IP.
     - Enables SSH access for secure login.
     - Fully automated provisioning.

```
# Login to Azure
Connect-AzAccount

# Set Variables
$resourceGroup = "DevOpsRG"
$location = "EastUS"
$vmName = "DevOpsVM"
$vmSize = "Standard_B2s"
$image = "UbuntuLTS"
$adminUser = "azureadmin"
$password = ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force

# Create Resource Group
New-AzResourceGroup -Name $resourceGroup -Location $location

# Create Virtual Network & Subnet
$vnet = New-AzVirtualNetwork -ResourceGroupName $resourceGroup -Location $location -Name "DevOpsVNet" -AddressPrefix "10.0.0.0/16"
$subnet = Add-AzVirtualNetworkSubnetConfig -Name "DevOpsSubnet" -VirtualNetwork $vnet -AddressPrefix "10.0.1.0/24"
$vnet | Set-AzVirtualNetwork

# Create Public IP
$publicIp = New-AzPublicIpAddress -ResourceGroupName $resourceGroup -Location $location -Name "DevOpsPublicIP" -AllocationMethod Static

# Create Network Security Group
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $resourceGroup -Location $location -Name "DevOpsNSG"

# Allow SSH (Port 22)
$nsgRule = New-AzNetworkSecurityRuleConfig -Name "AllowSSH" -Description "Allow SSH" -Access Allow -Protocol Tcp -Direction Inbound -Priority 100 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 22
$nsg | Add-AzNetworkSecurityRuleConfig -NetworkSecurityRule $nsgRule | Set-AzNetworkSecurityGroup

# Create Network Interface
$nic = New-AzNetworkInterface -Name "DevOpsNIC" -ResourceGroupName $resourceGroup -Location $location -SubnetId $subnet.Id -PublicIpAddressId $publicIp.Id -NetworkSecurityGroupId $nsg.Id

# Create VM
$cred = New-Object System.Management.Automation.PSCredential ($adminUser, $password)
New-AzVM -ResourceGroupName $resourceGroup -Location $location -Name $vmName -Credential $cred -Size $vmSize -Image $image -NetworkInterfaceId $nic.Id
```

#### Secure Azure environment
- I take a layered approach to securing Azure: strong IAM with RBAC and MFA, isolated networking using NSGs and private endpoints, encrypted data, real-time monitoring with Defender for Cloud, and enforcement through Azure Policy. I also integrate security into CI/CD to shift left and prevent issues before they reach production

#### 
- 
----------------------------------------------------------------------


- Define role instance in Azure.
- What is  Azure Service Level Agreement (SLA)
- What is Azure virtual machine scale sets
- What do you understand about cloud computing
- What are the various models available for cloud deployment
- What is NSG
- What do you understand about the Availability Set
- What are the available options for deployment environments provided by Azure
- What do you need to do when drive failure occurs
- What is transient fault handling
- What does transient error mean
- What should you configure your application to do in order to handle transient errors
- What is azure storage key
- What is cspack in Azure
- What is the best Azure solution for executing the code without a server
- What would be the best feature recommended by Azure for having a common file sharing system between multiple virtual machines
- What are the differences between Azure Scale Sets and Availability Sets
- What is Azure Scale Sets
- What is Azure Availability Sets
- What would happen when the maximum failed attempts are reached during the process of Azure ID Authentication
- What is Azure Blob Storage
- What is Storage Account
- What is Block blobs
- What is Append blobs
- What is Page blobs
- What do you understand by Azure Scheduler
- What feature of Azure can be used to stop the issue of high load on the application in cases of no man support on the flow
- What are the types of storage services apart from blob storage provided by Azure
- What are IaaS, PaaS and SaaS
- What are the differences between the Azure Table Storage and the Azure SQL service
- What is Azure Table Storage
- What is Azure SQL service
- What are the differences between the Azure Storage Queue and the Azure Service Bus Queue
- What is Azure Storage Queue
- What is Azure Service Bus Queue
- What is dead letter
- What are the possible causes of the client application to be disconnected from the cache
- What are the different types of services offered in the cloud
- What is Public Cloud
- What is Private Cloud
- What is Hybrid Cloud
- What is Microsoft Azure and why is it used
- What are Roles and why do we use them
- What is VM  Role
- Why is Azure Diagnostics API needed
- How many cloud service roles are provided by Azure
- How can a VM be created by means of Azure CLI
- Can you tell something about Azure Cloud Service
- I have some private servers on my premises, also I have distributed some of my workload on the public cloud, what is this architecture called
- Is it possible to design applications that handle connection failure in Azure
- Is it possible to login to a Linux Virtual Machine without using a password
- Is it possible to get a public DNS or IP address for the Azure Internal Load Balancer
- Is it possible to map the Windows machines running on two different port numbers, say 80 and 81, on an IIS Web Server to an Azure Load Balancer
- VM creation is possible using Azure Resource Manager in a Virtual Network which was created by means of classic deployment. True or False
- You have an application running on the On-Prem Server and have backup on Azure East US region. Now, On-Prem server application access fails. Is it possible to access the application via the Azure environment
- What is Cloud Computing
- What is Microsoft Azure
- What is Azure as PaaS
- What are Break-fix issues in Microsoft Azure
- What is the main difference between the repository and the powerhouse server
- What are unconnected lookups
- What is the use of the Migration Assistant tool in Azure Websites
- What is the use of Azure Active Directory
- What is HDInsight in Microsoft Azure
- What are the important drawbacks of using Microsoft Azure
- What is MOSS
- What is the step you need to perform when drive failure occurs
- What it's the difference between PROC MEANS and PROC SUMMARY
- What is the use of VNET
- What the important requirements when creating a new Virtual Machine
- What are the three main components of the Windows Azure platform
- What is the purpose of using an application partition scheme in Azure
- What do you mean by the network security groups
- What happens when you exhaust the maximum failed attempts by authenticating yourself using Azure AD
- What is the use of Temp Drive in VM
- What is csrun
- What is Azure Cloud Service
- What are the roles implemented in Windows Azure
- What are the three principal segments of Windows Azure platform
- What is the distinction between Windows Azure Queues and Windows Azure Service Bus Queues
- What is autoscaling in Azure
- What are the features of Windows Azure
- What are the differences between a public cloud and a private cloud
- What is table storage in Windows Azure
- What is Windows Azure Portal
- What do you comprehend about Hybrid Cloud
- What is a storage key
- What is Windows Azure Traffic Manager
- What is federation in SQL Azure
- What is SQL Azure database
- What are the different types of Storage areas in Windows Azure
- What is the concept of the table in Windows Azure
- What is TFS build system in Azure
- What is Azure App Service
- What is profiling in Azure
- What is cmdlet in Azure
- What is Windows Azure Scheduler
- What is Text Analytics API in Azure Machine
- What is Migration Assistant tool in Azure Websites
- What is the distinction between Public Cloud and Private Cloud
- What is Azure Service Level Agreement (SLA)
- What are the different types of services offered in the cloud
- What are the different cloud deployment models
- What is Microsoft Azure and why is it used
- What are Roles and why do we use them
- What are virtual machine scale sets in Azure
- What is an Availability Set
- What are Network Security Groups
- What is a break-fix issue
- What happens when you exhaust the maximum failed attempts for authenticating yourself via Azure AD
- What is a VNet
- What are the differences between Subscription Administrator and Directory Administrator
- What is the difference between Service Bus Queues and Storage Queues
- What is Azure Redis Cache
- What are Redis databases
- What are the username requirements when creating a VM
- What are the password requirements when creating a VM
- What are the various power states of a VMPower States - Azure Interview Questions - Edureka
- What is Azure Search
- What are the expected values for the Startup File section when I configure the runtime stack
- What is the difference between price, software price, and total price in the cost structure for Virtual Machine offers in the Azure Marketplace
- What are stateful and stateless microservices for Service Fabric
- What is the meaning of application partitions
- What are special Azure Regions
- What is meant by Microsoft Azure and Azure diagnostic
- What is meant by cloud computing
- What is the scalability of the cloud computing
- What are the advantages of cloud computing
- What is meant by PaaS, SaaS, and IaaS
- What are the main functions of the Azure Cloud Service
- What do you mean by roles
- What are the different types of roles
- What do you mean by a domain Update Domains Fault Domain
- What do you mean by a BLOB and what are their types
- What is meant by the block blob and page BLOB
- What is meant by the DeadLetter queue
- What are the sizes of the Azure VM
- What is meant by table storage
- What is meant by the enterprise warehousing
- What do you mean by lookup transformation
- What is meant by the connected lookups
- What is meant by the unconnected lookups
- What is meant by the command task
- What are the PowerCenter commands that can be used in Informatica
- What is the difference between copy and shortcut
- What do you mean by a service fabric in Azure
- What are the benefits of the traffic manager in Windows Azure
- What is the difference between a library and a list
- What do you mean by SAS
- What Are The Three Main Components Of Windows Azure Platform
- What Are The Service Model In Cloud Computing
- What Is Windows Azure Platform
- What Are The Roles Available In Windows Azure
- What Is Difference Between Windows Azure Platform And Windows Azure
- What Are The Three Types Of Roles In Compute Component In Windows Azure
- What Is Windows Azure Compute Emulator
- What Is Fabric
- What Are The Options To Manage Session State In Windows Azure
- What Is Cspack
- What Is Guest Os
- What Is Web Role In Windows Azure
- What Is The Difference Between Public Cloud And Private Cloud
- What Is Windows Azure Diagnostics
- What Is Blob
- What Is The Difference Between Block Blob Vs Page Blob
- What Is The Difference Between Windows Azure Queues And Windows Azure Service Bus Queues
- What Is Deadletter Queue
- What Are Instance Sizes Of Azure
- What Is Table Storoage In Windows Azure
- What Is Azure Fabric Controller
- What Is Autoscaling
- What Is Vm Role In Windows Azure
- What Is A Cloud Service Role
- What Is Link A Resource 
- What Is Scale A Cloud Service 
- What Is A Web Role 
- What Is A Worker Role 
- What Is A Role Instance 
- What Is A Guest Operating System 
- What Is A Cloud Service Components
- What Is Deployment Environments
- What Is A Service Definition File
- What Is A Service Configuration File
- What Is A Service Package 
- What Is A Cloud Service Deployment 
- What Is Azure Diagnostics 
- What Is Azure Service Level Agreement (sla) 
- What is the link to a resource
- What is scale a cloud service
- What is a web role
- What is a worker role
- What is a role instance
- What is a guest operating system
- What is a cloud service component
- What is a service package
- What is a cloud service deployment
- What is Azure Diagnostics
- What is Azure Service Level Agreement (SLA.
- What is Azure Active Directory and how it is used
- What is the use of a Lookup transformation 
- What is the difference between Azure Service Bus ueues and Storage ueues
- What is the Windows Azure Platform
- What is the difference between the Windows Azure Platform and Windows Azure
- What are the options to manage session state in Windows Azure  
- What is the guest OS
- What is the dead letter queue
- What is swap deployments
- What is minimal vs. verbose monitoring
- What are the instance sizes of Azure
- What is Diagnostics
- What is command task
- What is Cmdlet command
- What is role instance
- What is service fabric
- What is Availability Set
- What is service definition file
- What is enterprise warehousing
- What is lookup transformation
- What is cspack
- What is Azure Service Level Agreement
- What is the concept of the table
- What is guest OS
- What is Azure Fabric.
- What are the different deployment model of the cloud
- What is Window Azure platform
- What is traffic manager benefits in Azure
- What is Azure Resource Manager
- What is Azure Service Fabric.
- What are the types of services you can build with the Service Fabric.
- What is log analytics
- Name some important applications of Microsoft Azure
- Name the types of web application which can be deployed with Azure
- Name the services which are used to manage resources in Azure
- Name various power states of a Virtual Machine.
- Name two blobs used in Microsoft Azure
- Name three types of Disks used by VMs
- Name two types of cloud services
- Name the web application types that can be deployed with the Azure
- Explain the Importance of the role and how many types of roles are available in Windows Azure
- Explain the crucial benefits of Traffic Manager
- Why should you use Azure CDN
- Why is Azure Active Directory used
- Why doesnt Azure Redis Cache have an MSDN class library reference like some of the other Azure services
- Why was my client disconnected from the cache
- Which service in Azure is used to manage resources in Azure
- Which of the following web applications can be deployed with Azure
- Which services are used to manage the resources in Azure
- How does a character analytics API function
- How many customers subscriptions allowed in managed disks
- How much storage can a user with a virtual machine use
- How can you create an HDInsight Cluster in Azure
- How can I use applications with Azure AD that Im using on-premises
- How much storage can I use with a virtual machine
- How can one create a Virtual Machine in Powershell
- How to create a new storage account and container using Power Shell
- How can one create a VM in Azure CLI
- How can you retrieve the state of a particular VM
- How can you stop a VM using Power Shell
- How are Azure Marketplace subscriptions priced
- How is the price of the Azure subscription placed
- How is Azure Resource Manager beneficial over the classic services
- How Many Types Of Deployment Models Are Used In Cloud
- How Many Instances Of A Role Should Be Deployed To Satisfy Azure Sla (service Level Agreement)  And Whats The Benefit Of Azure Sla
- How To Programmatically Scale Out Azure Worker Role Instances
- How Would You Categorize Windows Azure (iaas/paas/saas)
- How many instances of a Role should be deployed to satisfy Azure SLA (service level agreement. And whats the benefit of Azure SLA
- How to programmatically scale-out Azure Worker Role instances
- How to design applications to handle connection failure in Windows Azure
- When will you find the list of built-in app with ADD
- Where can I find a list of applications that are pre-integrated with Azure AD and their capabilities
- Difference Between Web And Worker Roles In Windows Azure
- Difference - PROC SUMMARY and PROC MEANS
- Difference - Repetitive vs Minimal monitoring.
- Difference copy vs shortcut
- Difference - Library vs List
- Define Windows Azure AppFabric.
- Define the Azure Redis Cache.
- A __Web_______ role is a virtual machine instance running Microsoft IIS Web server that can accept and respond to HTTP or HTTPS requests.
- Apart From .net Framework, Name Other Three Language/framework That Can Be Used To Develop Windows Azure Applications
- Are data disks provide support within scale sets
- Are data disks supported within scale sets
- Are there any scale limitations for customers using managed disks
- Can you create VM by using Microsoft Azure Resource Manager in a Virtual Network
- Compare the STS and SPS and state its important features
- Describe the common architecture of SharePoint 2010
- Differentiate between Microsoft Azure and AWS.
- Differentiate between the verbose and minimal monitoring.
- Differentiate between the Windows Azure bus queues and Windows Azure queues
- Differentiate between the repository and the powerhouse server
- Discuss the different database types in SQL Azure
- Do scale sets work with Azure availability sets
- Enlist the monitoring features that are present in the SharePoint 2010
- Give a clear overview of API in Azure
- I have some private servers on my premises, also I have distributed some of my workload on the public cloud, what is this architecture called
- If the client gets disconnected from cache with the services state the probable cause
- Is it possible to create a Virtual Machine using Azure Resource Manager in a Virtual Network that was created using classic deployment
- Is it possible to add an existing VM to an availability set
- My web app still uses an old Docker container image after Ive updated the image on Docker Hub. Does Azure support continuous integration/deployment of custom containers
- State the difference pricing model of Microsoft Azure
- State the purpose of the cloud configuration file
- State the class that can be used to retrieve data
- State some features of SAS
- State what will you do in case of a drive failure
- State what should be done in case of a service failure
