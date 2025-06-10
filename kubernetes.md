
#### k8s Kubernetes Architecture
- its a **master nodes and worker nodes** concept
- Kubernetes is a cluster system, it is  used to automate the deployment, scaling, and management of containerized applications. 
- **Master Node (Control Plane)**: The master node is like the brain of the cluster. It controls and manages the entire system.
     - **API Server**		: The communication hub that exposes the Kubernetes API for all interactions.
     - **Scheduler**		: Decides where to deploy your application’s containers based on available resources.
     - **Controller Manager**	: Ensures the desired state of the system is maintained, such as managing scaling and node health.
     - **Etcd**			: A distributed database that stores all configuration and state data of the cluster.
- **Worker Nodes (Data Plane)**: Worker nodes are the machines where your actual applications run. Each node is responsible for running containers.
     - **Kubelet**		: Ensures that containers are running as expected.
     - **Kube-proxy**		: Manages network routing and load balancing for the services.
     - **Container Runtime**	: The software (like Docker) that runs the actual containers.
- The master node manages and monitors everything, while the worker nodes actually run the workloads.
- This architecture allows Kubernetes to provide automated scaling, self-healing, and high availability for your applications.

- indetail
     - **Kubelet** manages containers on a node and communicates with the API server. It gets notified when a pod is scheduled and instructs the container runtime (e.g., Docker) to pull and run images.
     - **Kube-proxy** is a service proxy that load-balances traffic and manages routing rules to forward requests to backend pods.
     - **Container runtime**  pulls images and runs containers based on specifications.
     - **Pods** are the smallest deployable units in Kubernetes, similar to lightweight VMs. They can contain one or more containers and are ephemeral, receiving new IPs each time they restart.
     - **kubectl** is a command-line tool for interacting with the Kubernetes cluster, supporting both imperative and declarative management approaches.

1. Workload Objects (Pod & Controllers)
     - **Pod** – The smallest deployable unit, consisting of one or more containers.
     - **ReplicaSet** – Ensures a specified number of pod replicas are running.
     - **Deployment** – it manages ReplicaSets for rolling updates and rollbacks.
     - **StatefulSet** – it manages stateful applications, ensuring stable network identities and persistent storage.
     - **DaemonSet** – Ensures a pod runs on every node (e.g., logging and monitoring agents).
     - **Job** – Runs a batch job and ensures it completes successfully.
     - **CronJob** – Schedules jobs to run at specific times, similar to cron jobs in Linux.
2. Service & Networking Objects
     - **Service** – Exposes a set of pods as a network service (ClusterIP, NodePort, LoadBalancer, ExternalName).
     - **Ingress** – it mean traffic entering the Kubernetes cluster from an external source.
     - **Egress** - Egress is traffic leaving the Kubernetes cluster (e.g., accessing external services like APIs, databases, or internet ).
     - **Endpoint** – Represents network endpoints for a service.
     - **NetworkPolicy** – Controls traffic flow between pods.
3. Configuration & Storage Objects
     - **ConfigMap** – Stores configuration data as key-value pairs.
     - **Secret** – Stores sensitive information like passwords and API keys securely.
     - **PersistentVolume (PV)** – Represents storage in the cluster.
     - **PersistentVolumeClaim (PVC)** – Requests specific storage from a PV.
     - **StorageClass** – Defines dynamic storage provisioning.
4. Cluster Management Objects
     - **Namespace** – Provides isolation between groups of resources in a cluster.
     - **Node** – Represents a worker machine in the cluster.
     - **Role & RoleBinding** – Defines and applies permissions within a namespace.
     - **ClusterRole & ClusterRoleBinding** – Assigns permissions across the cluster.
     - **ServiceAccount** – Provides identity for processes running in a pod.

#### Kubectl commands
-  it is the command-line interface (CLI) for interacting with a Kubernetes cluster. It allows you to deploy applications, inspect resources, manage configurations, and troubleshoot issues.
1. Basic Commands
     - kubectl version  ( Show kubectl and API server versions )
     - kubectl cluster-info  	(Show cluster information )
     - kubectl config view  	(View kubeconfig settings)
     - kubectl get nodes	List all nodes in the cluster
     - kubectl describe node <node-name>	Show detailed info about a node
2. Context and Configuration
     - kubectl config get-contexts	List available contexts
     - kubectl config current-context	Show the current context
     - kubectl config use-context <context>	Switch to a different context
     - kubectl config set-context <context>	Set a new context
3. Workloads: Pods, Deployments, and ReplicaSets
     - kubectl get pods	List all pods
     - kubectl get pods -o wide	List pods with extra details
     - kubectl describe pod <pod-name>	Show detailed pod information
     - kubectl logs <pod-name>	View logs of a pod
     - kubectl logs -f <pod-name>	Follow live logs
     - kubectl exec -it <pod-name> -- /bin/sh	Open a shell inside a pod
     - kubectl delete pod <pod-name>	Delete a specific pod
     - kubectl delete pods --all	Delete all pods
     - kubectl rollout status deployment <deployment-name>	Check deployment rollout status
     - kubectl rollout restart deployment <deployment-name>	Restart a deployment
4. Creating and Managing Resources
     - kubectl apply -f <file>.yaml	Apply configuration from a file
     - kubectl create deployment <name> --image=<image>	Create a deployment
     - kubectl delete -f <file>.yaml	Delete resources from a file
     - kubectl scale deployment <deployment-name> --replicas=<number>	Scale a deployment
     - kubectl edit deployment <deployment-name>	Edit a deployment resource
5. Services and Networking
     - kubectl get svc	List all services
     - kubectl describe svc <service-name>	Show detailed service information
     - kubectl expose deployment <deployment-name> --type=NodePort --port=<port>	Expose a deployment as a service
     - kubectl port-forward <pod-name> <local-port>:<pod-port>	Forward a local port to a pod
6. ConfigMaps and Secrets
     - kubectl get configmaps	List all ConfigMaps
     - kubectl describe configmap <configmap-name>	Show ConfigMap details
     - kubectl create configmap <name> --from-literal=key=value	Create a ConfigMap
     - kubectl get secrets	List all Secrets
     - kubectl describe secret <secret-name>	Show secret details
7. Namespaces
     - kubectl get namespaces	List all namespaces
     - kubectl create namespace <namespace>	Create a new namespace
     - kubectl delete namespace <namespace>	Delete a namespace
8. Persistent Volumes and Storage
     - kubectl get pv	List Persistent Volumes (PV)
     - kubectl get pvc	List Persistent Volume Claims (PVC)
9. Debugging and Troubleshooting
     - kubectl describe <resource> <name>	Detailed resource information
     - kubectl logs <pod-name>	View logs
     - kubectl exec -it <pod-name> -- /bin/sh	Open a shell inside a pod
     - kubectl get events	Show cluster events
     - kubectl top pod	Show CPU and memory usage for pods
     - kubectl top node	Show CPU and memory usage for nodes
10. Advanced Commands
     - kubectl apply -k <directory>	Apply a Kustomization
     - kubectl diff -f <file>.yaml	Show differences before applying changes
     - kubectl explain <resource>	Show resource documentation
     - kubectl label pod <pod-name> key=value	Add a label to a pod
     - kubectl annotate pod <pod-name> key=value	Add an annotation to a pod

#### CronJob
- A CronJob automatically runs tasks at specified time intervals, such as:
     - Running database backups every night.
     - Generating reports at scheduled times.
     - Sending emails or notifications periodically.
     - Cleaning up logs or temporary files at regular intervals.
- CronJob Work
     - The CronJob Controller schedules and triggers jobs based on the cron schedule.
     - Each time the schedule is met, Kubernetes creates a Job, which then creates a Pod to run the task.
     - After the Pod completes its execution, it terminates.
     - Failed jobs can be retried depending on the configuration.
Use Cases
     - Automated Backups - Periodically back up databases or persistent data.
     - Log Rotation & Cleanup - Remove old logs and free up disk space.
     - Data Processing - Periodically process batch jobs (e.g., analytics, reporting).
     - Health Checks & Alerts - Run periodic checks on system health and send alerts.
     - Periodic Sync Jobs - Synchronize data between different systems or databases.



#### Kubelet
Kubelet is a critical node-level agent in Kubernetes that ensures containers are running in a Pod. It communicates with the Kubernetes API server, manages container lifecycles, and monitors health.
     - Registers the node with the Kubernetes API server.
     - Starts and stops containers inside Pods.
     - Manages Pod lifecycle and ensures containers run as expected.
     - Monitors Pod health using liveness & readiness probes.
     - Communicates with container runtime (Docker, containerd, CRI-O) to manage containers.
- **Role of Kubelet in Kubernetes**
     - The kubelet is a node agent that manages pod lifecycle, health checks, and container runtime interactions.
     - It ensures that the desired state of Pods matches the actual state
- How does Kubelet interact with the container runtime
     - Kubelet uses the Container Runtime Interface (CRI) to interact with container runtimes like Docker, containerd, CRI-O.
     - It sends commands to start, stop, and monitor containers.
- Kubelet process stops
     - The node will stop reporting to the Kubernetes API server.
     - Pods running on that node may become unreachable.
     - controller manager will schedule new pods on available nodes if configured properly.

#### Pods
- it is the smallest deployable unit in Kubernetes that runs one or more containers with shared networking and storage.
- **Pod Architecture & Components**
     - Containers: Main application + sidecar containers.
     - Network: Every Pod gets a unique IP and can communicate with other Pods.
     - Storage Volumes: Shared storage between containers in a Pod.
- What happens when a Pod fails
     - If managed by a Deployment/ReplicaSet, Kubernetes will reschedule it.
     - If a standalone Pod, it must be recreated manually.
- Pod have multiple containers
     - multiple containers in a Pod can share resources and work together (e.g., an app container + a logging sidecar).
- How do Pods communicate with each other
     - Each Pod gets a unique IP address.
     - Pods communicate within the cluster via Kubernetes networking (CNI plugins like Calico, Flannel, Cilium).
- info
     - Simplifi ed application management
     - Improved scaling and availability
     - Easy deployment and rollback
     - Improved resource utilizatio
     - Increased portability and fl exibility
     - A pod is the smallest deployable unit in Kubernetes that represents a single instance of a running process in a container.

- Conclusion
     - Pods are the smallest deployable unit in Kubernetes.
     - Single-container Pods run simple applications, while multi-container Pods use Sidecar patterns.
     - Pods are managed by controllers like Deployments, StatefulSets, and DaemonSets.
     - Networking, storage, and scheduling are important considerations when deploying Pods.

#### Pod Lifecycle
- Pods go through different phases during their lifecycle:
     - **Phase** -	Description
     - **Pending** - 	The Pod is created but waiting for resources.
     - **Running** -	At least one container is running.
     - **Succeeded** -	All containers have successfully completed execution.
     - **Failed** - 	One or more containers exited with an error.
     - **Unknown** -	The Pod state cannot be determined.

#### Pod Restart Policies
- A Pod's restart policy controls how Kubernetes handles failed containers:
     - **Always** -	Restarts the container if it fails (default).
     - **OnFailure** -	Restarts only if it fails with an error.
     - **Never** -	Does not restart the container.

#### How Pods Are Managed
- Pods are usually managed by Controllers:
     - **Deployment** → Scales and manages ReplicaSets.
     - **StatefulSet** → Manages stateful applications.
     - **DaemonSet** → Runs a Pod on every node.
     - **Job & CronJob** → Runs Pods for batch processing.

#### Pod Networking
- details
     - Each Pod gets a unique IP.
     - Containers within a Pod share the same network namespace.
     - Communication between Pods happens using ClusterIP Services.

#### pod pending
- The node has a taint, which prevents scheduling unless the pod has a matching toleration.
     - we can add a toleration in the pod spec (if pod should run on tainted node)
     - we can remove the taint from the node (if not needed)
     - we can Use nodeSelector or nodeAffinity instead of taints (if controlling placement)

- The pod is stuck in a pending state and is not getting scheduled.
     - Insufficient resources in the cluster
     - NodeSelector, Taints, or Affinity rules preventing scheduling
     - PVC (Persistent Volume Claim) is not bound
          -  kubectl describe pod <pod-name>
          -  kubectl describe nodes
          -  kubectl get pvc

#### taint
it is a rule applied to a Kubernetes node that prevents certain pods from being scheduled on it unless the pod has a matching toleration

#### StatefulSet
- StatefulSet is a Kubernetes controller used to manage stateful applications that require:
     - Stable network identities (e.g., pod-0, pod-1, pod-2).
     - Persistent storage that remains even if the Pod is restarted.
     - Ordered and controlled scaling (both up and down).
     - Common Use Cases:
          - Databases (MySQL, PostgreSQL, MongoDB).
          - Message Queues (Kafka, RabbitMQ).
          - Distributed Applications (Elasticsearch, Zookeeper)
- it is a resource object used to manage the deployment of stateful applications. It provides guarantees for ordering and uniqueness of pods, allowing each pod to have a stable hostname and persistent storage. StatefulSets are often used for databases, messaging systems, or other applications that require stable network identities and persistent data.

#### ClusterIP
- it is internal communication between pods within the cluster. It assigns an internal IP and is only accessible inside the cluster.
- It's used for backend services, databases, and internal APIs. For external access, LoadBalancer, NodePort, or Ingress is used.

#### Namespaces 
- Shortcode = ns
- Namespaces are virtual clusters within a Kubernetes cluster.
     - Help organize resources in multi-team or multi-project environments.
     - Allow resource isolation, RBAC, and quotas.
- Example: You can have dev, staging, and prod namespaces with separate resources.
- default namespaces
     - default
     - kube-system
     - kube-public
     - kube-node-lease
- namespaces provide a mechanism for isolating groups of resources within a single cluster. Names of resources need to be unique within a namespace, but not across namespaces.
     - **kubectl create namespace <namespace_name>**	To create a namespace by the given name.
     - **kubectl get namespace**	To list the current namespace in a cluster.
     - **kubectl describe namespace <namespace_name>**	To display the detailed state of one or more namespaces.
     - **kubectl delete namespace <namespace_name>**	To delete a namespace.
     - **kubectl edit namespace <namespace_name>**	To edit and update the definition of a namespace.

#### ExternalName
- it maps a service name to an external DNS instead of routing traffic to internal pods.
- It’s used to access external services like third-party APIs or databases using Kubernetes DNS, without exposing cluster resources.

#### Deployment
- Deployment is a Kubernetes object used to manage stateless applications.
- Functions:
     - Automatically creates and manages ReplicaSets.
     - Ensures the desired number of pods are running.
     - Supports rolling updates, rollbacks, and scaling.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
```

- its a higher-level resource object that allows you to declaratively define and manage the lifecycle of a set of replica pods. It provides a way to ensure the desired number of pod replicas are running and allows for rolling updates and rollbacks of the application.
- A Deployment in Kubernetes manages the rollout and scaling of Pods. It ensures:
     - Declarative updates to applications.
     - Automated rollbacks if failures occur.
     - Scaling of replicas based on demand.
     - Rolling updates & zero downtime deployments.
     - Use Cases:
          - Deploying stateless applications (e.g., Nginx, API services).
          - Managing multiple replicas for high availability.
          - Performing rolling updates & rollbacks safely.
- Deployment Works
     - Define a Deployment (specifies the desired state).
     - Kubernetes creates and manages ReplicaSets, ensuring the correct number of Pods run.
     - Handles updates & rollbacks automatically to prevent downtime.
     - Uses labels and selectors to group Pods efficiently.
- A Deployment provides declarative updates for Pods and ReplicaSets.The typical use case of deployments are to create a deployment to rollout a ReplicaSet, declare the new state of the pods and rolling back to an earlier deployment revision.
     - **kubectl create deployment <deployment_name>**	To create a new deployment.
     - **kubectl get deployment**	To list one or more deployments.
     - **kubectl describe deployment <deployment_name>**	To list a detailed state of one or more deployments.
     - **kubectl delete deployment<deployment_name>**	To delete a deployment.

#### Edit a deployment and change the image
- When you change the container image in a deployment:
     - Kubernetes performs a rolling update:
          - It gradually terminates old pods.
          - Simultaneously creates new pods with the updated image.
     - Ensures zero downtime (if configured properly).
- **Example**: Updating from nginx:1.25 to nginx:1.26 triggers a rollout of new pods using the new image.

#### Delete deployment
- steps
     - The associated ReplicaSet and all its pods are deleted.
     - Services or ConfigMaps referring to the deployment won’t work anymore unless reconfigured.
     - The deployment configuration itself is removed from the cluster.
- Note: Persistent volumes (PVCs) are not deleted unless specifically defined in the pod spec with persistentVolumeReclaimPolicy: Delete.

#### Service types
- ClusterIP (default): Internal access within the cluster only.
- NodePort: Exposes the service on each node's IP at a static port.
- LoadBalancer: Provisions an external load balancer (cloud providers).
- ExternalName: Maps a service to an external DNS name.

#### External - internal service
- Internal: A backend service used only by frontend pods.
- External: An API exposed to customers via LoadBalancer.

|Feature|Internal Service|External Service|
|-|-|-|
|Access Scope|Inside cluster only|Accessible from outside|
|Service Type|ClusterIP|NodePort / LoadBalancer|
|Use Case|Microservice communication|User-facing endpoints|

#### Stateful & Stateless
- **stateful application** maintains data or session state, like databases or message queues. Kubernetes uses resources like StatefulSets and PersistentVolumes to manage stateful applications, ensuring stable network identities and storage persistence across pod restarts or rescheduling
     - Stateful apps retain data and require persistent storage. They use StatefulSets, each pod has a stable identity and its own storage. Example: databases like MySQL.

- **stateless application** does not store any data or client session information between requests. It can be easily scaled horizontally because all replicas behave the same — examples include web servers or front-end services
     - Stateless apps don’t store data between requests. They use Deployments, are easy to scale, and all pods are identical. Example: web servers.

#### Stateful
- they are applications store and maintain data between requests. Unlike stateless applications, they require persistent storage and cannot be easily replaced without losing data.
- Key Characteristics of Stateful Applications:
     - Data is stored and maintained across sessions.
     - Requires persistent storage (databases, logs, etc.).
     - Scaling is complex due to data consistency needs.
     - Failure of a node may require data recovery or replication.
- Examples of Stateful Applications:
     - Databases: MySQL, PostgreSQL, MongoDB.
     - Message Queues: Kafka, RabbitMQ.
     - Storage Systems: ElasticSearch, Cassandra.
     - Applications with user sessions: Banking apps, e-commerce carts.

#### Stateless
- Stateless applications do not store data or session state on the server. Each request is independent and does not rely on previous requests.
- Key Characteristics:
     - No persistent storage needed.
     - Each request is handled separately.
     - Can be easily scaled horizontally (add more replicas).
     - Failure of a single instance does not affect the overall system.
- Examples of Stateless Applications:
     - Web servers (Nginx, Apache).
     - RESTful APIs.
     - Microservices.
     - Containers running ephemeral workloads.

#### Microservices
- its an architectural style where an application is built as a collection of small, independent services that communicate over a network (usually via APIs).
- Each microservice is responsible for a specific business functionality and can be developed, deployed, and scaled independently
     - Large-Scale Applications: If your application is complex and needs independent scaling.
     - Frequent Deployments: When you need CI/CD pipelines and faster releases.
     - Multiple Development Teams: Ideal when teams work on separate services.
     - Cloud & Container-Based Deployments: Best suited for Kubernetes, Docker, AWS Lambda.
     - Business Domain Separation: When different parts of your system have distinct business logic.

#### Nodeport
- it exposes a service on a static port on all nodes in the cluster. This allows external traffic to access the service using NodeIP:NodePort.
- NodePort Works
     - A NodePort service selects a pod using labels.
     - Kubernetes assigns a static port (default range: 30000–32767).
     - The service becomes accessible via http://<NodeIP>:<NodePort>.
     - Any request to this port is forwarded to the underlying pods

#### Node
- its the fundamental compute unit in a Kubernetes cluster where containers run. Nodes are worker machines (can be physical or virtual) that execute Pods.
     - Master node: Runs control plane components.
     - Worker node: Runs pods (application containers).
- Each node contains essential services to run containers, such as:
     - Kubelet – Communicates with the control plane and manages pods.
     - Container Runtime – Runs containerized applications (Docker, containerd, CRI-O).
     - Kube Proxy – Handles networking and service communication.

#### Service Mesh
- A Service Mesh is an infrastructure layer that manages service-to-service communication in microservices, enhancing security, observability, and reliability without changing application code. It abstracts networking complexities and ensures seamless service interactions.
- its a dedicated networking layer that helps manage communication between microservices in a Kubernetes cluster.
- It provides traffic control, security, observability, and reliability without modifying the application code
- 𝗔𝗿𝗰𝗵𝗶𝘁𝗲𝗰𝘁𝘂𝗿𝗲
     - **Data Plane** – Handles the actual traffic flow between services, typically using sidecar proxies like Envoy.
     - **Control Plane** – Manages policies, security, service discovery, and observability for better control and governance.
- 𝗞𝗲𝘆 𝗕𝗲𝗻𝗲𝗳𝗶𝘁𝘀
     - Traffic Control – Load balancing, failover, retries
     - Security – Mutual TLS (mTLS), authentication, authorization, zero-trust security
     - Observability – Distributed tracing, logging, metrics for debugging and monitoring
     - Resilience – Circuit breaking, rate limiting, retries, fault injection
     - Multi-cluster & Multi-cloud Support – Connects services across Kubernetes clusters and hybrid cloud environments
- 𝗣𝗼𝗽𝘂𝗹𝗮𝗿 𝗦𝗲𝗿𝘃𝗶𝗰𝗲 𝗠𝗲𝘀𝗵𝗲𝘀
     - Istio – Feature-rich, Kubernetes-native, integrates well with K8s RBAC and policies
     - Linkerd – Lightweight, easy to use, lower resource footprint
     - HashiCorp Consul – Multi-platform, supports VM and Kubernetes environments, strong service discovery capabilities
     - Kuma – Built on Envoy, supports both Kubernetes and VMs, good for multi-cloud setups
     - AWS App Mesh – Fully managed, deeply integrated with AWS services

- 𝗪𝗵𝗲𝗻 𝘁𝗼 𝗨𝘀𝗲 𝗮 𝗦𝗲𝗿𝘃𝗶𝗰𝗲 𝗠𝗲𝘀𝗵?
     - Large-scale microservices architectures with complex service-to-service communication
     - Need for end-to-end security with zero-trust networking (mTLS, authorization)
     - Deep observability for debugging, monitoring, and tracing service interactions
     - Managing multi-cluster or hybrid cloud deployments
     - Requirement for advanced traffic policies like canary deployments, blue-green deployments
- A service mesh in Kubernetes is a dedicated infrastructure layer that handles communication between services in a microservices architecture. 
     - It provides advanced networking features such as load balancing, service discovery, traffic management, security, and observability. 
     - By injecting a sidecar proxy into each pod, a service mesh enables fine-grained control and monitoring of service-to-service communication without requiring changes to the application code.

#### Stateless vs. Stateful

|Feature|Stateless|Stateful|
|- |-|-|
|Data Storage|No persistent storage|Uses persistent storage|
|Session Handling|Each request is independent|Requires session tracking|
|Scaling|Easily scalable|More complex to scale|
|Resilience|Any instance can handle requests|Requires coordination between instances|
|Use Case|Web servers, APIs, microservices|Databases, Kafka, Zookeeper|

#### PV - Persistent Volumes
- those are storage resource in Kubernetes that provides persistent storage to applications, ensuring that data remains available even if a Pod restarts or is rescheduled to another node.
- those are storage resources in Kubernetes that exist independently of Pods. They provide a way to retain data even if a Pod is deleted, rescheduled, or crashes.
     - **Decouples storage from Pods** – Data persists beyond Pod lifecycles.
     - **Supports dynamic provisioning** – Automatically creates storage on demand.
     - **Multiple storage backends** – AWS EBS, Azure Disk.
     - **Access modes define how PVs are used** – ReadWriteOnce, ReadOnlyMany, ReadWriteMany.

- A PersistentVolume in Kubernetes is a cluster-wide resource that represents a piece of network-attached storage in a cluster. 
     - It provides a way to provision and manage persistent storage that can be used by pods.
     - PersistentVolumes decouple storage from individual pods, allowing storage to persist beyond the lifecycle of pods. 
     - They can be dynamically provisioned or statically configured, providing a unifi ed interface for persistent storage in Kubernetes.

#### PVC - Persistent Volume Claims
- They are requests for storage by pods
- they are like request for storage by a Pod in Kubernetes. It allows applications to use persistent storage that remains intact even if a Pod is restarted or rescheduled.
     - A PVC requests storage from a Persistent Volume (PV).
     - Ensures data persistence across Pod restarts and rescheduling.
     - Works with different storage backends ( EBS, NFS, Azure Disk ).
     - Uses Storage Classes for dynamic volume provisioning.
- A PersistentVolumeClaim (PVC) is a request for a specific amount of storage resources from a PersistentVolume (PV). It allows Pods to use persistent storage in a decoupled manner

#### Kubernetes errors and fix

#### CrashLoopBackOff
1. **CrashLoopBackOff** - The container repeatedly crashes after starting.
     - Application misconfiguration
     - Missing dependencies
     - Insufficient resources (CPU/Memory)
     - Improper readiness/liveness probes
     - The container exits immediately after execution
          - kubectl logs <pod-name> -n <namespace>
          - kubectl describe pod <pod-name> -n <namespace>
          - Increase CPU/memory limits if necessary.
          - Ensure application handles errors properly.
- **CrashLoopBackOff in a Production Pod**
- Issue: A critical application pod was stuck in a CrashLoopBackOff state. Logs showed an OutOfMemoryError.
     - Increased memory requests/limits in the pod spec.
     - Optimized JVM heap size (for Java app).
     - Used kubectl logs and kubectl describe pod to analyze failure causes.
     - Deployed with a resource monitoring policy.
       
#### ImagePullBackOff / ErrImagePull** 
- when Kubernetes cannot pull the container image from the registry.
     - Incorrect image name or tag
     - Authentication issues with private registry
     - Image does not exist in the registry
          - Verify the image name and tag
          - If using a private registry, ensure correct credentials:
          - Restart pod after fixing the issue.
- **ImagePullBackOff Due to Private Registry Authentication Issues**
- Issue: New deployment failed with ImagePullBackOff due to missing authentication credentials for a private container registry.
     - Verified registry access (kubectl get secrets).
     - Recreated the imagePullSecret and added it to the service account.
     - Updated deployment YAML to reference the correct secret.
     - Used kubectl describe pod to confirm successful pull.
       
#### OOMKilled (Out of Memory)
- The container exceeds its allocated memory limit.
     -  Insufficient memory allocated in the pod spec.
     -  A memory leak in the application.
          - Increase memory limits in the pod YAML
          - kubectl top pod <pod-name> -n <namespace>

#### NodeNotReady
- The node is not in a ready state.
     - Node running out of resources
     - Network connectivity issues
     - Kubernetes components (kubelet, etc.) not running properly
          - kubectl get nodes
          - kubectl describe node <node-name>
          - systemctl restart kubelet
          - journalctl -u kubelet -f
          
#### CreateContainerConfigError
- The container cannot be created due to incorrect configurations.
     - Incorrect environment variables
     - Secret or ConfigMap missing
     - Volume mount issues
          -  kubectl describe pod <pod-name>
          - kubectl get configmap
          - kubectl get secret
            
#### CreateContainerError
- Kubernetes fails to start the container.
     - Problems with the container runtime
     - Insufficient privileges
     - Host path issues
          - journalctl -u containerd -f

#### DeadlineExceeded
- A job or request takes too long and gets terminated.
     - Timeout in readiness/liveness probes
     - Network issues preventing completion
          - kubectl get svc

#### BackOff
- Kubernetes retries starting a container but fails repeatedly.
     - Failed dependencies
     - Readiness probe failures
          - kubectl get events --sort-by=.metadata.creationTimestamp

#### Unauthorized / Forbidden Errors
- Permissions issue with Kubernetes RBAC. 
     - ServiceAccount lacks required roles
     - Missing ClusterRole or Role bindings
          - kubectl auth can-i <action> <resource>
          - kubectl get rolebinding -A

#### PersistentVolumeClaim (PVC) Stuck in Pending
- Issue: An application required persistent storage, but the PVC was stuck in Pending.
     - Checked storage class (kubectl get storageclass).
     - Found that no available PersistentVolumes matched the PVC.
     - Created a new PersistentVolume and bound it manually.
     - Automated PVC creation via Helm to avoid misconfigurations.

#### High Latency in Services Due to Improper HPA
- Issue: A microservice was experiencing high response times under load, even though Horizontal Pod Autoscaler (HPA) was enabled.
     - Investigated using kubectl top pods and kubectl describe hpa.
     - Found that CPU-based scaling was configured, but the application was memory-intensive.
     - Changed HPA to scale based on memory and custom application metrics.
     - Improved readinessProbe to remove unresponsive pods from load balancer rotation.

#### Node Failure (Production Incident)
- Issue: One of the worker nodes in the cluster went NotReady, causing some workloads to become unavailable.
     - Used kubectl get nodes and kubectl describe node to check node status.
     - Found an issue with disk space (Filesystem full).
     - Cleaned up unused Docker images and logs.
     - Restarted the kubelet service (systemctl restart kubelet).
     - Applied auto-recovery automation via node draining and re-provisioning.

#### Network Latency Between Services (Intermittent 5XX Errors)
- Issue: Microservices communication was unstable, leading to intermittent 502 and 504 errors.
     - Used kubectl exec to test connectivity and kubectl logs for debugging.
     - Found that istio sidecar injection was missing for some pods.
     - Re-deployed affected services with Istio sidecar enabled.
     - Implemented a retry policy at the Istio virtual service level.

#### Ingress Controller Misconfiguration
- Issue: A new service was not accessible externally, and Ingress showed 404 Not Found.
     - Used kubectl describe ingress to check misconfiguration.
     - Found missing host field in ingress rules.
     - Updated the Ingress YAML with the correct path and host.
     - Restarted the Nginx Ingress Controller pod.

#### Secrets Management Issue (Application Failing to Start)
- Issue: An application failed to start due to missing environment variables that were sourced from Kubernetes Secrets.
     - Verified secrets using kubectl get secrets and kubectl describe secrets.
     - Found incorrect reference in deployment YAML.
     - Updated the deployment to use envFrom instead of manually specifying each variable.
     - Enabled automatic secret rotation via an external secrets manager.

#### HPA (Horizontal Pod Autoscaler)  & VPA ( Vertical Pod Autoscaler )
- **HPA (Horizontal Pod Autoscaler)** : when we set it automatically scale the number of pods in a deployment, replica set, or stateful set based on observed CPU/memory utilization or custom metrics.
- **VPA ( Vertical Pod Autoscaler )** : its a component that monitors resource consumption and automatically adjusts pod CPU and memory requests.
- Unlike Horizontal Pod Autoscaler (HPA), which scales the number of pods, VPA scales up or down the resources assigned to each pod

**HPA in detail :**
1. **Benifits:**
     - **Handles Dynamic Traffic** – Automatically adjusts pod count based on real-time demand.
     - **Improves Application Availability** – Prevents performance degradation during high loads.
     - **Optimizes Resource Costs** – Scales down pods when demand is low to save resources.
     - **Works with Custom Metrics** – Can scale pods based on application-specific metrics (e.g., request latency).
2. **How Does HPA Work**
     - HPA continuously monitors metrics and adjusts the number of pods accordingly:
     - **Metrics Collection** – HPA uses the Kubernetes Metrics Server or external/custom metrics to evaluate CPU, memory, or other parameters.
     - **Scaling Decision** – If resource usage exceeds the defined threshold, HPA adds more pods; if usage drops, it removes pods.
     - **Pod Scaling** – Kubernetes adjusts the number of running pods in the deployment.
3. **Best Practices for HPA**
     - Use with VPA – Combine HPA for pod scaling and VPA for resource tuning.
     - Define Proper Min/Max Limits – Prevent excessive scaling that may affect performance or cost.
     - Monitor Scaling Behavior – Use monitoring tools like Prometheus + Grafana.
     - Use Custom Metrics – Scale based on real-world application load (e.g., request count).
4. **When to Use HPA**
     - Web applications with fluctuating traffic.
     - Microservices that need dynamic scaling.
     - Event-driven applications where load varies based on external triggers.

**VPA in detail :**
1. **Benifits:**
     - **Optimized Resource Usage:** Ensures applications get only the necessary resources, reducing underutilization.
     - **Improved Performance:** Prevents CPU throttling and Out of Memory (OOM) issues by assigning appropriate resources.
     - **Reduced Manual Effort:** Automates resource allocation, reducing the need for manual tuning.
     - **Better Cost Management:** Prevents over-provisioning and minimizes cloud resource costs.
2. **How Does VPA Work**
- VPA operates in three main modes:
     -  **Off Mode** – Only provides recommendations without making actual changes.
     - **Auto Mode** – Automatically updates pod resource requests.
     - **Initial Mode** – Sets resource requests only at pod startup.
3. **VPA Components**
     - **VPA Recommender**: Analyzes pod resource usage and suggests CPU/memory adjustments.
     - **VPA Updater:** Reschedules pods with updated resource limits.
     - **VPA Admission Controller:** Adjusts resource requests for newly created pods.
4. **Best Practices for Using VPA**
     - **Use VPA with HPA:** Combine VPA for resource tuning and HPA for scaling pods.
     -  **Enable VPA in "Off" Mode First:** Start with recommendations before enabling auto mode.
     - **Monitor Pod Restart Impact:** Since VPA may restart pods, be cautious with stateful applications.
     - **Use Custom Metrics:** Fine-tune scaling policies based on real-world application behavior.
5. **When to Use VPA**
     - For stateful applications (databases, caching services) that need controlled resource scaling.
     - When optimizing CPU and memory allocation for cost efficiency.
     - In microservices environments, where resource usage varies across services.

#### States and Components (kubectl get all)
- When you run kubectl get all, you get an overview of various Kubernetes resources in the current namespace, including:
- **Core Components Displayed by kubectl get all**

| Component	| Description |
| ---------- |----------------------------------------- |
|Pod	| The smallest deployable unit in Kubernetes that runs a container or multiple containers. |
|Deployment	| Manages a set of identical pods, ensuring high availability and rolling updates. |
|ReplicaSet	| Ensures a specified number of pod replicas are running at all times. |
|StatefulSet |	Manages stateful applications, providing stable network IDs and persistent storage. |
|DaemonSet	| Ensures a copy of a pod runs on each (or some) nodes (e.g., logging or monitoring agents). |
|Service	| Exposes a set of pods as a network service. |
|Ingress	| Manages external access to services, usually via HTTP/HTTPS. |
|Job	| Runs a task to completion, ensuring it runs a specified number of times. |
|CronJob |	Runs scheduled jobs at specified intervals. |

- **Key Fields in kubectl get all Output** - General Resource Fields

|Field	| Description |
| ---------- |----------------------------------------- |
|Name	| The name of the resource (Pod, Deployment, Service, etc.).|
|Type	| The type of resource (e.g., ClusterIP, NodePort, LoadBalancer for services).|
|Cluster-IP |	Internal IP of a service within the cluster (if None, it means it's a headless service). |
|External-IP	| IP accessible from outside the cluster (only for LoadBalancer or NodePort services).|
|Ports |	Ports exposed by the service or pod (e.g., 9093, 9094, 8080, etc.).|
|Age	| The duration since the resource was created. |

 - **Deployment & ReplicaSet Specific Fields**

|Field	| Description |
| ---------- |----------------------------------------- |
|Desired |	Number of replicas expected in a Deployment or ReplicaSet.|
|Current	| Number of replicas currently scheduled.|
|Ready	| Number of replicas that are successfully running and ready to serve traffic. |
|Up-to-Date |	Number of replicas updated to the latest version.|
|Available |	Number of replicas available to serve traffic.|

#### Ingress
- info
     - A Kubernetes ingress is an API object that manages external access to services within a cluster.
     - It acts as a configurable entry point that routes incoming traffi c to diff erent services based on defi ned rules and policies.
- info 2
     - Ingress is a Kubernetes object that defines HTTP and HTTPS routing rules for external traffic.
     - Routes traffic based on hostnames, paths, headers, etc.
     - Can handle TLS termination and rewrite rules.
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: webapp-service
            port:
              number: 80

```

Ingress Architecture - it is to manage external access to services within a cluster.
- It provides a way to configure rules for routing HTTP and HTTPS traffi c to diff erent services.
     - Ingress Controller: Handles external traffic (e.g., Nginx, Traefik)
     - Ingress Resource: Defines routing rules (ingress.yaml)
     - Service: Exposes your application inside Kubernetes
     - Pods: Run your application

#### Ingress Controller
- An Ingress Controller is the actual component that implements the routing rules defined in Ingress objects.
     - Examples: NGINX Ingress Controller, HAProxy, Traefik, AWS ALB Controller.
     - Must be installed separately; Kubernetes does not include one by default.
- its a component in Kubernetes that manages external access to services inside a cluster, typically via HTTP/HTTPS. It routes requests based on hostnames, paths, or other rules
     - Single Entry Point – Manages all external traffic
     - Path-Based Routing – Routes /api to one service, /web to another
     - SSL Termination – Manages HTTPS for all services
     - Load Balancing – Distributes traffic between pods
     - Authentication & Security – Enables OAuth, JWT, and IP whitelisting
- use cases
     - Host multiple applications behind one external IP.
     - Route requests based on host/path (e.g., foo.com/api, foo.com/app).
     - Handle TLS termination.
     - Implement authentication, rate limiting, rewrites, etc.
- Example Use Case: A company hosts:
     - api.company.com → routes to microservice A
     - app.company.com → routes to frontend UI All behind a single NGINX Ingress.

#### Ingress Default Backend
- its a fallback service that handles requests which do not match any Ingress rule.
     - It typically returns a 404 Not Found or custom error page.
     - It prevents dropped or misrouted traffic.
- **Example**: If a user accesses app.example.com but no Ingress rule matches it, the request is routed to the default backend.

- It specifies what to do with an incoming request to the Kubernetes cluster that isn't mapped to any backend i.e what to do when no rules being defined for the incoming HTTP request If the default backend service is not defined, it's recommended to define it so that users still see some kind of message instead of an unclear error


#### YAML - JSON
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: mycontainer
      image: nginx
```

#### Egress
Egress is traffic leaving the Kubernetes cluster (e.g., accessing external services like APIs, databases, or internet ).

#### Maintenance activity on the K8 node
- Whenever there are security patches available the Kubernetes administrator has to perform the maintenance task to apply the security patch to the running container in order to prevent it from vulnerability, which is often an unavoidable part of the administration. The following two commands are useful to safely drain the K8s node.
- **kubectl cordon**
- **kubectl drain –ignore-daemon set**

The first command moves the node to maintenance mode or makes the node unavailable, followed by kubectl drain which will finally discard the pod from the node. After the drain command is a success you can perform maintenance.

Note: If you wish to perform maintenance on a single pod following two commands can be issued in order:
- **kubectl get nodes** : to list all the nodes
- **kubectl drain <node name>**: drain a particular node

#### Central logs from POD
- This architecture depends upon the application and many other factors. Following are the common logging patterns
     - Node level logging agent.
     - Streaming sidecar container.
     - Sidecar container with the logging agent.
     - Export logs directly from the application.
- In the setup, journalbeat and filebeat are running as daemonset. Logs collected by these are dumped to the kafka topic which is eventually dumped to the ELK stack.
- The same can be achieved using EFK stack and fluentd-bit

#### Monitor the cluster
- Prometheus is used for Kubernetes monitoring. The Prometheus ecosystem consists of multiple components.
     - Mainly Prometheus server which scrapes and stores time-series data.
     - Client libraries for instrumenting application code.
     - Push gateway for supporting short-lived jobs.
     - Special-purpose exporters for services like StatsD, HAProxy, Graphite, etc.
     - An alert manager to handle alerts on various support tools.

#### Various things that can be done to increase security
- By default, POD can communicate with any other POD, we can set up network policies to limit this communication between the PODs.
- RBAC (**Role-based access control**) to narrow down the permissions.
- Use namespaces to establish security boundaries.
- Set the admission control policies to avoid running the privileged containers.
- Turn on audit logging.

#### Load Balancing
Load Balancing is one of the most common and standard ways of exposing the services. There are two types of load balancing in K8s and they are:
- **Internal load balancer** – This type of balancer automatically balances loads and allocates the pods with the required incoming load.
- **External Load Balancer** – This type of balancer directs the traffic from the external loads to backend pods.

Load balancing is a way to distribute the incoming traffic into multiple backend servers, which is useful to ensure the application available to the users.
![image](https://github.com/user-attachments/assets/f152bc61-1d80-436d-a5bb-7dddd4d7d093)
In Kubernetes, as shown in the above figure all the incoming traffic lands to a single IP address on the load balancer which is a way to expose your service to outside the internet which routes the incoming traffic to a particular pod (via service) using an algorithm known as round-robin. Even if any pod goes down load balances are notified so that the traffic is not routed to that particular unavailable node. Thus load balancers in Kubernetes are responsible for distributing a set of tasks (incoming traffic) to the pods.

#### init container and when it can be used
init containers will set a stage for you before running the actual POD.
     - Wait for some time before starting the app Container with a command like sleep 60.
     - Clone a git repository into a volume.

#### PDB (Pod Disruption Budget)
A Kubernetes administrator can create a deployment of a kind: PodDisruptionBudget for high availability of the application, it makes sure that the minimum number is running pods are respected as mentioned by the attribute minAvailable spec file. This is useful while performing a drain where the drain will halt until the PDB is respected to ensure the High Availability(HA) of the application. The following spec file also shows minAvailable as 2 which implies the minimum number of an available pod (even after the election).
- Example: Y**AML Config using minAvailable => **

#### K8's services running on node
- Mainly K8 cluster consists of two types of nodes, **master and executor**.

- **Master services**:
     - **Kube-apiserver:** Master API service which acts as an entry point to K8 cluster.
     - **Kube-scheduler:** Schedule PODs according to available resources on executor nodes.
     - **Kube-controller-manager:**  is a control loop that watches the shared state of the cluster through the apiserver and makes changes attempting to move the current state towards the desired stable state
- **Executor node:** (This runs on master node) Worker Node
     - **Kube-proxy:** This service is responsible for the communication of pods within the cluster and to the outside network, which runs on every node. This service is responsible to maintain network protocols when your pod establishes a network communication.
     - **kubelet:** Each node has a running kubelet service that updates the running node accordingly with the configuration(YAML or JSON) file. NOTE: kubelet service is only for containers created by Kubernetes.

#### Control the resource usage of POD
- With the use of limit and request resource usage of a POD can be controlled.
     - **Request:** The number of resources being requested for a container. If a container exceeds its request for resources, it can be throttled back down to its request.
     - **Limit:** An upper cap on the resources a single container can use. If it tries to exceed this predefined limit it can be terminated if K8's decides that another container needs these resources. If you are sensitive towards pod restarts, it makes sense to have the sum of all container resource limits equal to or less than the total resource capacity for your cluster.

#### GKE
GKE is Google Kubernetes Engine that is used for managing and orchestrating systems for Docker containers. With the help of Google Public Cloud, we can also orchestrate the container cluster.

#### Operator
As an extension to K8, the operator provides the capability of managing applications and their components using custom resources. Operators generally comply with all the principles relating to Kubernetes, especially those relating to the control loops
- purpose of operators
As compared to stateless applications, achieving desired status changes and upgrades are handled the same way for every replica, managing Kubernetes applications is more challenging. The stateful nature of stateful applications may require different handling for upgrading each replica, as each replica might be in a different state. Therefore, managing stateful applications often requires a human operator. This is supposed to be assisted by Kubernetes Operator. Moreover, this will pave the way for a standard process to be automated across several Kubernetes clusters.

#### Why should namespaces be used - namespace cause problems
Over the course of time, using the default namespace alone is proving to be difficult, since you are unable to get a good overview of all the applications you can manage within the cluster as a whole. The namespaces allow applications to be organized into groups that make sense, such as a namespace for all monitoring applications and another for all security applications. 
- Additionally, namespaces can be used for managing Blue/Green environments, in which each namespace contains its own version of an app as well as sharing resources with other namespaces (such as logging or monitoring). It is also possible to have one cluster with multiple teams using namespaces. The use of the same cluster by multiple teams may lead to conflict.  Suppose they end up creating an app that has the same name, this means that one team will override the app created by the other team as Kubernetes prohibits two apps with the same name (within the same namespace).

#### Troubleshoot if the POD is not getting scheduled
In K8’s scheduler is responsible to spawn pods into nodes. There are many factors that can lead to unstartable POD. The most common one is running out of resources, use the commands like kubectl describe <POD> -n <Namespace> to see the reason why POD is not started. Also, keep an eye on kubectl to get events to see all events coming from the cluster.

#### Forward the port
Forward the port '8080 (container) -> 8080 (service) -> 8080 (ingress) -> 80 (browser) and how it can be done
- The ingress is exposing port 80 externally for the browser to access, and connecting to a service that listens on 8080. The ingress will listen on port 80 by default. An "ingress controller" is a pod that receives external traffic and handles the ingress and is configured by an ingress resource For this you need to configure the ingress selector and if no 'ingress controller selector' is mentioned then no ingress controller will manage the ingress.

#### Different ways to provide external network connectivity to K8
- By default, POD should be able to reach the external network but vice-versa we need to make some changes. Following options are available to connect with POD from the outer world.
     - **Nodeport** (it will expose one port on each node to communicate with it)
     - **Load balancers** (L4 layer of TCP/IP protocol)
     - **Ingress** (L7 layer of TCP/IP Protocol)
- Another method is to use Kube-proxy which can expose a service with only cluster IP on the local system port.
     - $ **kubectl proxy --port=8080 $ http://localhost:8080/api/v1/proxy/namespaces//services/:/**

#### Run a POD on a particular node
- Various methods are available to achieve it.
     - **nodeName:** specify the name of a node in POD spec configuration, it will try to run the POD on a specific node.
     - **nodeSelector:** Assign a specific label to the node which has special resources and use the same label in POD spec so that POD will run only on that node.
     - **nodeaffinities:** required DuringSchedulingIgnoredDuringExecution, preferredDuringSchedulingIgnoredDuringExecution are hard and soft requirements for running the POD on specific nodes. This will be replacing nodeSelector in the future. It depends on the node labels.

#### differences between Docker Swarm and Kubernetes
Below are the main difference between Kubernetes and Docker:
- The installation procedure of the K8s is very complicated but if it is once installed then the cluster is robust. On the other hand, the Docker swarm installation process is very simple but the cluster is not at all robust.
- Kubernetes can process the auto-scaling but the Docker swarm cannot process the auto-scaling of the pods based on incoming load.
- Kubernetes is a full-fledged Framework. Since it maintains the cluster states more consistently so autoscaling is not as fast as Docker Swarm.

#### run Kubernetes locally
Kubernetes can be set up locally using the Minikube tool. It runs a single-node bunch in a VM on the computer. Therefore, it offers the perfect way for users who have just ongoing learning Kubernetes.

#### default range of ports used to expose a NodePort service
30000-32767

#### Kubernetes Terminology
- Terms that you should be familiar with before starting off with Kubernetes are enlisted below:
     - **Cluster**	It can be thought of as a group of physical or virtual servers where Kubernetes is installed.
     - **Nodes**     There are two types of Nodes, 
     - **Master**    node is a physical or virtual server that is used to control the Kubernetes cluster.
     - **Worker**     node is the physical or virtual server where workload runs in given container technology.
     - **Pods**     The group of containers that shares the same network namespaces.
     - **Labels**     These are the key-value pairs defined by the user and associated with Pods.
     - **Master**	It controls plane components to provide access points for admins to manage the cluster workloads.
     - **Service**	It can be viewed as an abstraction that serves as a proxy for a group of Pods performing a "service".

#### Nodes:  ShortCode = no
- A Node is a worker machine in Kubernetes and may be either a virtual or a physical machine, depending on the cluster. Each Node is managed by the control plane. A Node can have multiple pods, the Kubernetes control plane automatically handles scheduling the pods across the Nodes in the cluster.
     - **kubectl get node**	To list down all worker nodes.
     - **kubectl delete node <node_name>**	Delete the given node in cluster.
     - **kubectl top node**	Show metrics for a given node.
     - **kubectl describe nodes | grep ALLOCATED -A 5**	Describe all the nodes in verbose.
     - **kubectl get pods -o wide | grep <node_name>**	List all pods in the current namespace, with more details.
     - **kubectl get no -o wide**	List all the nodes with mode details.
     - **kubectl describe** n     - Describe the given node in verbose.
     - **kubectl annotate node <node_name>**	Add an annotation for the given node.
     - **kubectl uncordon node <node_name>**	Mark my-node as schedulable.
     - **kubectl label node**	Add a label to given node

#### Pods : Shortcode = po
- Pods are the smallest deployable units of computing that you can create and manage in Kubernetes.
     - **kubectl get po**	To list the available pods in the default namespace.
     - **kubectl describe pod <pod_name>**	To list the detailed description of pod.
     - **kubectl delete pod <pod_name>**	To delete a pod with the name.
     - **kubectl create pod <pod_name>**	To create a pod with the name.
     - **Kubectl get pod -n <name_space>**	To list all the pods in a namespace.
     - **Kubectl create pod <pod_name> -n <name_space>**	To create a pod with the name in a namespace.

#### Services 
- A Service in Kubernetes is an abstraction that defines a stable network endpoint for accessing a set of Pods. Since Pods are ephemeral (they can be created or destroyed dynamically), a Service ensures that applications can communicate reliably with other Pods.
- Kubernetes Service Needed?
     - Pods have dynamic IP addresses, meaning:
          - A Pod’s IP changes when it restarts.
          - Other Pods or external users cannot reliably connect to a Pod.
- A Service provides a fixed IP (ClusterIP) and a DNS name for a group of Pods, ensuring stable communication.

- Best Practices for Using Services
     - Use ClusterIP for internal communication between microservices.
     - Use Ingress for HTTP/HTTPS routing instead of NodePort for better flexibility.
     - Monitor services with Prometheus & Grafana to track health and traffic.
     - Use NetworkPolicies to restrict access between services.
 - commands
     - **kubectl get services**	To list one or more services.
     - **kubectl describe services <services_name>**	To list the detailed display of services.
     - **kubectl delete services -o wide**	To delete all the services.
     - **kubectl delete service < service_name>**	To delete a particular service.

#### DaemonSets
- its workload controller that ensures that a specific pod runs on every node within a cluster. 
- useful for deploying background services or agents that need to be present on each node, such as logging collectors, monitoring agents, or networking components.
- they automatically schedule and maintain pods on new nodes that are added to the cluster and remove them from nodes that are removed, ensuring consistent pod distribution across the cluster.
- A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected. Deleting a DaemonSet will clean up the Pods it created.
     - **kubectl get ds**	To list out all the daemon sets.
     - **kubectl get ds -all-namespaces**	To list out the daemon sets in a namespace.
     - **kubectl describe ds [daemonset_name][namespace_name]**	To list out the detailed information for a daemon set inside a namespace.

#### Events
- Kubernetes events allow us to paint a performative picture of the clusters.
     - **kubectl get events**	To list down the recent events for all the resources in the system.
     - **kubectl get events --field-selector involvedObject.kind != Pod**	To list down all the events except the pod events.
     - **kubectl get events --field-selector type != Normal**	To filter out normal events from a list of events.

#### Logs
- Logs are useful when debugging problems and monitoring cluster activity. They help to understand what is happening inside the application.
     - **kubectl logs <pod_name>**	To display the logs for a Pod with the given name.
     - **kubectl logs --since=1h <pod_name>**	To display the logs of last 1 hour for the pod with the given name.
     - **kubectl logs --tail-20 <pod_name>**	To display the most recent 20 lines of logs.
     - **kubectl logs -c <container_name> <pod_name>**	To display the logs for a container in a pod with the given names.
     - kubectl logs <pod_name> pod.log	To save the logs into a file named as pod.log.

#### ReplicaSets
- A ReplicaSet's purpose is to maintain a stable set of replica Pods running at any given time. As such, it is often used to guarantee the availability of a specified number of identical Pods.
     - **kubectl get replicasets**	To List down the ReplicaSets.
     - **kubectl describe replicasets <replicaset_name>**	To list down the detailed state of one or more ReplicaSets.
     - **kubectl scale --replace=[x]** To scale a replica set.

#### Service Accounts
- its an identity used by Pods to authenticate with the Kubernetes API. Unlike user accounts (which represent humans), ServiceAccounts are used by applications or services running inside the cluster.
- By default, every Pod runs with the default ServiceAccount in its namespace unless another one is specified.
- Why Use ServiceAccounts
     - Secure authentication for Pods when interacting with the Kubernetes API.
     - Assign fine-grained RBAC permissions to restrict access.
     - Enable Pods to pull container images from private registries.
     - Used by controllers, monitoring agents, and CI/CD tools to manage workloads securely.
- Best Practices for ServiceAccounts
     - Use custom ServiceAccounts instead of the default one.
     - Grant only necessary permissions using RBAC (Principle of Least Privilege).
     - Avoid assigning cluster-admin to a ServiceAccount.
     - Rotate ServiceAccount tokens for better security.
     - Restrict external access by controlling how Pods use ServiceAccounts.
- Conclusion
     - ServiceAccounts provide authentication for Pods to interact with the Kubernetes API.
     - By default, Pods use the default ServiceAccount unless specified otherwise.
     - RBAC (Roles & RoleBindings) must be assigned to grant permissions.
     - Best practices involve restricting permissions and using custom ServiceAccounts for applications

- A service account provides an identity for processes that run in a Pod.
- Commands
     - **kubectl get serviceaccounts**	To List Service Accounts.
     - **kubectl describe serviceaccounts**	To list the detailed state of one or more service accounts.
     - **kubectl replace serviceaccounts**	To replace a service account.
     - **kubectl delete serviceaccounts <name>**	To delete a service account.

#### Changing Resource Attributes
- Taints: They ensure that pods are not placed on inappropriate nodes.
     - **kubectl taint <node_name><taint_name>**	This is used to update the taints on one or more nodes.
- Labels: They are used to identify pods.
     - **kubectl label pod <pod_name>** 	Add or update the label of a pod

#### Cluster Introspection
-  Details
     - **kubectl version**	To get the information related to the version.
     - **kubectl cluster-info**	To get the information related to the cluster.
     - **kubectl config g view**	To get the configuration details.
     - **kubectl describe node <node_name>**	To get the information about a node.

#### API Server (kube-apiserver)
- The API Server is the central control plane component in Kubernetes, acting as the gateway for all interactions with the cluster. It validates, processes, and serves API requests from users, controllers, and other components.
- **Key Responsibilities of API Server**
- Handles All API Requests
     - The API server exposes the Kubernetes API over HTTP(S) and serves as the frontend for the control plane.
     - All interactions with the cluster (e.g., kubectl, UI, automation) go through the API server.
- Authentication & Authorization
     - It enforces authentication (Token, OAuth, Client Certificates, etc.).
     - Implements authorization mechanisms like RBAC, ABAC, and Webhook authorizers.
- Validates & Processes Requests
     - Ensures API requests are well-formed and conform to Kubernetes schema (using OpenAPI).
     - Performs request validation before persisting data.
- Acts as the Gatekeeper to etcd
     - Persists and retrieves cluster state from etcd, ensuring strong consistency.
     - The API server is the only component that directly communicates with etcd.
- Admission Control & Policy Enforcement
     - Uses admission controllers to enforce security, resource limits, and best practices.
     - Examples: Namespace limits, Pod Security Admission, ResourceQuota.
- Handles API Aggregation & Extensions
     - Supports Custom Resource Definitions (CRDs) and API Aggregation Layer to extend Kubernetes APIs

#### Interacting with Deployments and Services
- details
     - **kubectl logs deploy/my-deployment**	Dump Pod logs for a Deployment (single-container case).
     - **kubectl logs deploy/my-deployment -c my-contain**	dump Pod logs for a Deployment (multi-container case).
     - **kubectl port-forward svc/my-service 5000**	To listen on local port 5000 and forward to port 5000 on Service backend.
     - **kubectl port-forward deploy/my-deployment 5000:6000**	To listen on local port 5000 and forward to port 6000 on a Pod created by <my-deployment>.
     - **kubectl exec deploy/my-deployment -- ls**	To run command in first Pod and first container in Deployment (single- or multi-container cases).

#### Copy files and directories to and from containers
- details
     - **kubectl cp /tmp/foo_dir my-pod:/tmp/bar_dir**	Copy /tmp/foo_dir local directory to /tmp/bar_dir in a remote pod in the current namespace.
     - **kubectl cp /tmp/foo my-pod:/tmp/bar -c my-container**	Copy /tmp/foo local file to /tmp/bar in a remote pod in a specific container.
     - **kubectl cp /tmp/foo my-namespace/my-pod:/tmp/bar**	Copy /tmp/foo local file to /tmp/bar in a remote pod in a specific container.
     - **kubectl cp my-namespace/my-pod:/tmp/foo /tmp/bar**	Copy /tmp/foo from a remote pod to /tmp/bar locally.

#### How can containers within a pod communicate with each other
- In Kubernetes, each Pod has an IP address. A Pod can communicate with another Pod by directly addressing its IP address, but the recommended way is to use Services. 
     - A Service is a set of Pods, which can be reached by a single, fixed DNS name or IP address. 

#### If a node is tainted is there a way to still schedule the pods to that node
- By default, The pods don't get scheduled when a node is tainted, however, you can start applying tolerations to the pod spec using the command
     - **kubectl taint nodes node1 key=value:NoSchedule**

#### init container
- details
     - The init containers will set a stage for you before running the actual POD.
     - Wait for some time before starting the app Container with a command like sleep 60.
     - Clone a git repository into a volume.

#### Node Autoscaling
- Node Autoscaling in Kubernetes dynamically adjusts the number of worker nodes in a cluster based on resource demands. It ensures cost optimization and availability by scaling up when workloads need more capacity and scaling down when resources are underutilized.
- Kubernetes provides two main ways to scale nodes:
     - Cluster Autoscaler (CA) – Automatically adjusts the number of nodes in the cluster.
     - Autoscaling with Cloud Provider (EKS, AKS, GKE, etc.) – Uses cloud-managed autoscaling features.
- Why Use Node Autoscaling
     - Cost Optimization – Scale down unused nodes to save costs.
     - High Availability – Ensure enough nodes for workloads during peak demand.
     - Self-Healing – If nodes are removed, workloads get rescheduled automatically.
     - Efficient Resource Utilization – Right-size infrastructure without manual intervention.
- The Cluster Autoscaler (CA) is a Kubernetes component that automatically adjusts the size of a cluster based on resource requests and scheduling needs.
     - Scale-Up: If pods are in the Pending state due to insufficient node resources, CA adds more nodes.
     - Scale-Down: If nodes are underutilized for a set period, CA removes them.
     - CA works at the node group level (e.g., AWS ASG, Azure VMSS, GCP Instance Groups).

#### Security Hardening
- Kubernetes security hardening ensures that your cluster is protected from attacks, misconfigurations, and unauthorized access. Kubernetes is a powerful orchestrator, but default settings are not always secure. Implementing security best practices reduces risks such as:
     - Unauthorized access (RBAC misconfigurations, weak authentication)
     - Container escape vulnerabilities (Pod privilege escalation)
     - Network security risks (open ports, unrestricted communication)
     - Data breaches (improper secrets management, insecure storage)
- Best Practices Summary
     - **API Server Security** -	Restrict API access, enable RBAC, audit logs
     - **RBAC** -	Use least privilege, avoid cluster-admin for apps
     - **Network Security**	- Apply Network Policies, disable public API access
     - **Pod Security** 	- Enforce PSA, drop root privileges, restrict host access
     - **Image Security** -	Use signed images, scan images for vulnerabilities
     - **Secret Management** -	Store secrets securely, encrypt at rest
     - **Logging & Monitoring** -	Enable audit logs, runtime monitoring (Falco, ELK)
- Conclusion
     - Security hardening is a continuous process, not a one-time setup.
     - Implement strong RBAC, Network Policies, and Pod Security Contexts.
     - Monitor, audit, and log all Kubernetes activities.
     - Use external secret management for sensitive data.
     - Automate security checks within CI/CD pipelines.

#### Performance Optimization
- it ensures that workloads run efficiently, resources are utilized effectively, and applications scale smoothly without unnecessary overhead. Below are the key areas to focus on for optimizing Kubernetes performance.
- Checklist
     - **API Server Performance** -	Reduce API calls, enable caching, optimize etcd
     - **Scheduler Tuning** 	- Affinity rules, topology spread constraints
     - **Worker Node Performance** -	Use efficient instance types, enable HugePages
     - **Pod & Container Resources** -	Set CPU/memory limits, enable VPA
     - **Autoscaling** -	Use HPA, Cluster Autoscaler, and KEDA
     - **Storage Performance** -	Use SSDs, optimize filesystem settings
     - **Networking Optimization** -	Enable CNI plugins with eBPF, use NodeLocal DNSCache
     - **Monitoring & Logging** -	Use Prometheus, Grafana, OpenTelemetry
     - **Deployment Optimization** -	Use lightweight images, enable caching, use probes
- Conclusion
     - Kubernetes performance optimization reduces costs, improves efficiency, and prevents downtime.
     - A combination of resource limits, autoscaling, network tuning, and monitoring is key.
     - Regular performance audits and fine-tuning keep clusters running optimally.

#### Disaster Recovery (DR)
- Disaster recovery (DR) in Kubernetes is essential to ensure cluster resilience in case of:
     - etcd data corruption or failure
     - Accidental deletion of resources
     - Cluster node failures
     - Network outages
     - Security incidents (e.g., ransomware, misconfigurations)
- Best Practices for Disaster Recovery
     - Automate Backups – Schedule periodic etcd & volume backups
     - Store Backups in Remote Storage – AWS S3, Azure Blob
     - Test Recovery Procedures – Conduct DR drills regularly
     - Monitor etcd Health – Use Prometheus & Grafana for monitoring
     - Use Infrastructure as Code (IaC) – Terraform/Helm to quickly recreate resources

#### Performance tuning
- Kubernetes performance optimization ensures efficient resource usage, faster deployments, and stable workloads. Here’s a structured guide on how to optimize your Kubernetes clusters for high performance and scalability.
- Best Practices for Kubernetes Performance Optimization
     - Use multi-zone clusters for high availability
     - Optimize etcd performance with proper memory & storage tuning
     - Use InitContainers for preloading dependencies instead of bloating main containers
     - Limit resource-heavy daemonsets (e.g., logging, monitoring agents)
     - Use gRPC instead of REST for inter-service communication

#### Custom Controller
- A Custom Controller is a user-defined control loop that monitors and manages resources in a Kubernetes cluster. It extends Kubernetes functionality beyond built-in controllers like Deployments, Jobs, and Services.
     - Continuously watches Kubernetes API for changes
     - Manages custom and built-in resources
     - Implements reconciliation logic to maintain the desired state
     - Automates complex operational tasks
- Custom Controller Work  - A custom controller follows the same pattern as native Kubernetes controllers:
     - Listens to API Server for events (watch, list, update, delete)
     - Processes changes and determines if action is needed
     - Reconciles the desired state by creating/updating resources
     - Loops continuously to monitor the cluster state
- Example - Custom Controller Flow - Let’s say you need a controller that automatically creates a ConfigMap when a new Pod is deployed.
     - The controller watches for new Pods in the cluster
     - When a Pod is created, it creates a ConfigMap dynamically
     - If the ConfigMap already exists, it ensures it matches the desired state
- Real-World Use Cases for Custom Controllers
     - Auto-scaling based on custom metrics (e.g., number of active users)
     - Enforcing security policies dynamically
     - Automated certificate renewal (e.g., Let’s Encrypt)
     - Self-healing applications by automatically restarting failed services

#### Custom schedulers
- A scheduler in Kubernetes is responsible for assigning Pods to Nodes based on available resources and constraints. By default, Kubernetes uses the default scheduler, but you can create a custom scheduler to implement your own scheduling logic.
- A custom scheduler is useful when you need specialized scheduling logic, such as:
     - Prioritizing Nodes with specific hardware (GPUs, high memory, etc.)
     - Affinity-based scheduling (grouping workloads on specific nodes)
     - Load balancing across multiple clusters
     - Scheduling based on business rules (e.g., cost optimization)
     - Latency-aware scheduling (placing workloads close to users)
- How Kubernetes Scheduling Works
- Default Scheduler: The default scheduler assigns Pods to Nodes based on:
     - Node availability and taints/tolerations
     - Pod affinity and anti-affinity
     - Resource requests (CPU, Memory)
- Custom Scheduler: A user-defined scheduler that:
     - Implements custom logic for pod placement
     - Runs as a separate deployment
     - Communicates with the Kubernetes API

-------------------------------------------

#### container networking
Kubernetes assigns a unique IP address to each Pod and allows communication between Pods using the Pod IP address. It sets up a virtual network through plugins like CNI to enable network connectivity.

#### Endpoint
- An Endpoint in Kubernetes represents the network location (IP + port) of a Pod that a Service routes traffic to. It is automatically created by Kubernetes when a Service selects Pods using a label selector.
- Key Points:
     - Endpoints store the actual IPs of Pods behind a Service.
     - Services use Endpoints to forward traffic to the correct Pods.
     - If a Service has no matching Pods, its Endpoint will be empty (i.e., no available backend).
- How Does an Endpoint Work?
     - A Service in Kubernetes selects Pods based on labels.
     - Kubernetes automatically creates an Endpoint object, listing the IP addresses of the matching Pods.
     - When a request reaches the Service, it is routed to one of the available Endpoints using round-robin or other load-balancing strategies.
     - If no Pods match the Service selector, the Service has no active Endpoints, and traffic cannot be forwarded.
- Best Practices
     - Always check if Endpoints exist when debugging a Service issue.
     - Use proper labels in Deployments to ensure Pods match the Service.
     - Consider using EndpointSlice for large-scale clusters.
     - Ensure Pods are in the ‘Ready’ state before expecting them in Endpoints.

#### StorageClass
- A StorageClass in Kubernetes provides a way to define and manage dynamic storage provisioning for Persistent Volumes (PVs). It allows administrators to specify different types of storage (e.g., SSD, HDD, NFS, cloud block storage) and their provisioning parameters.
- Why Use StorageClass?
     - Enables dynamic provisioning of Persistent Volumes (PVs).
     - Abstracts storage backends, making storage provisioning automated.
     - Supports different storage types (local, cloud, networked storage).
     - Allows users to define storage policies like performance, replication, and access modes.
     - Works across cloud providers (AWS EBS, Azure Disk, GCE Persistent Disk).
- How Does a StorageClass Work?
     - Define a StorageClass with a specific provisioner (e.g., AWS EBS, NFS, Ceph).
     - Create a Persistent Volume Claim (PVC) that references the StorageClass.
     - Kubernetes dynamically provisions a Persistent Volume (PV).
     - The PV is bound to the PVC and mounted inside a Pod.
- Best Practices for StorageClass
     - Choose the right storage type (SSD for performance, HDD for cost savings).
     - Set reclaim policy carefully (Retain for critical data, Delete for temporary storage).
     - Enable volume expansion (allowVolumeExpansion: true) for future scalability.
     - Use labels and annotations to organize StorageClasses.
     - Monitor storage usage with Prometheus & Grafana.
- Conclusion
     - StorageClass enables dynamic storage provisioning in Kubernetes.
     - Abstracts storage providers for flexible backend configurations.
     - Works with Persistent Volume Claims (PVCs) to request storage.
     - Supports cloud, on-prem, and network storage solutions.
     - Helps scale storage efficiently using expansion policies.

#### Role & RoleBinding
- they are components of Role-Based Access Control (RBAC) that manage permissions within a cluster.
     - **Role:** Defines permissions (what actions can be performed on resources).
     - **RoleBinding:** Assigns the Role to users, groups, or service accounts.
- Why Use Role & RoleBinding
     - Restricts access to resources within a namespace.
     - Enhances security by implementing least privilege principles.
     - Prevents unauthorized actions by users or applications.
     - Controls access at a granular level (e.g., read-only or full access).
- Conclusion
     - Role controls access at the namespace level, while ClusterRole is cluster-wide.
     - RoleBinding attaches Roles to users, groups, or service accounts.
     - RBAC improves security by enforcing access control policies.
     - Best practices involve least privilege, service accounts, and audits.

#### ClusterRole & ClusterRoleBinding
-  its a Kubernetes RBAC (Role-Based Access Control) resource that defines cluster-wide permissions for users, groups, or service accounts. Unlike a Role, which is namespace-scoped, a ClusterRole applies to all namespaces or cluster-level resources (e.g., nodes, persistent volumes).
- Why Use a ClusterRole?
     - Manages access to cluster-wide resources (e.g., nodes, storage, API access).
     - Provides permissions across multiple namespaces (unlike Role).
     - Used for built-in Kubernetes roles (e.g., admin, view, edit).
     - Essential for managing cluster operations and infrastructure components.
- Best Practices for ClusterRole
     - Use least privilege (grant only necessary permissions).
     - Prefer namespace-specific Roles unless cluster-wide access is needed.
     - Regularly review and audit ClusterRoles & ClusterRoleBindings.
     - Use groups instead of individual users for easier access management.
     - Limit cluster-admin access to prevent security risks.
- Conclusion
     - ClusterRole provides cluster-wide permissions for users or service accounts.
     - ClusterRoleBinding assigns ClusterRole to users, groups, or service accounts.
     - Essential for managing cluster-wide access control in Kubernetes.
     - Best practices involve using least privilege and regular audits.


#### scale applications
Applications in Kubernetes can be scaled horizontally by adjusting the number of Pod replicas, or vertically by changing the resource limits of individual Pods.

#### upgrade clusters
Kubernetes clusters can be upgraded by following the official upgrade guides provided by the Kubernetes project. The process involves upgrading control plane components and worker nodes.

#### expose a service outside the cluster
You can expose a service outside the Kubernetes cluster using a NodePort, LoadBalancer, or Ingress resource depending on the specifi c requirements of your application and infrastructure.

#### different types of volumes
Kubernetes supports various volume types, including EmptyDir, HostPath, PersistentVolumeClaim (PVC), ConfigMap, Secret.

#### secure access to API server
Access to the Kubernetes API server can be secured using authentication mechanisms like certifi cates, tokens, or external authentication providers. Role-Based Access Control (RBAC) can also be implemented to manage user access.

#### Helm chart
Helm is a package manager for Kubernetes. A Helm chart is a collection of files that describe a set of Kubernetes resources and their dependencies. It allows for easy installation and management of applications.

#### Horizontal Pod Autoscaler (HPA)
- A HorizontalPodAutoscaler automatically scales the number of Pod replicas based on CPU utilization or custom metrics. It ensures that an application can handle varying levels of traffic.

#### handle application configuration and sensitive information
- Sensitive information can be stored securely using Kubernetes Secrets. Application configuration can be managed using ConfigMaps or environment variables.

#### handle rolling back a failed deployment
-  Kubernetes allows you to roll back to a previous version of a Deployment by specifying the desired revision or using the kubectl rollout undo command. It reverts the Deployment to the previous state.

#### pod disruption budget (PDB)?
-  A Pod Disruption Budget defi nes the minimum number of Pods that must be available during a disruption caused by node maintenance or other events. It helps maintain application availability

#### container
- A container is a lightweight, standalone, executable software package that includes everything needed to run an application, including code, runtime, system tools, libraries, and settings.

#### components are included in the Kubernetes control plane?
The Kubernetes control plane consists of the following components:

#### etcd
- etcd is a distributed, consistent key-value store used by Kubernetes to store all cluster data. It serves as the source of truth for the entire cluster.
- **etcd is the heart of Kubernetes**. If it goes down, the cluster cannot function properly. Implement backup, security, and monitoring to ensure reliability.
- Example: When you create a pod using kubectl apply, the pod's configuration is stored in etcd.
     - Developed by CoreOS (now part of Red Hat).
     - Uses the Raft consensus algorithm to ensure strong consistency across nodes.
     - Kubernetes components (API Server, Controller Manager, Scheduler) communicate with etcd to store and retrieve cluster state.
- etcd Important - Stores all cluster configuration and state, including:
     - Nodes, Pods, Services, Deployments, ConfigMaps, Secrets
     - Role-based access control (RBAC) policies
     - Network configurations
- How Does etcd Work
     - etcd operates as a distributed system with multiple nodes (in HA setups)
     - It ensures that all cluster nodes have a consistent view of the data using the Raft Consensus Algorithm.
     - Leader Election: One node becomes the leader, while others are followers.
     - Writes go to the leader, which then replicates data to followers.
- etcd Architecture in Kubernetes
     - Runs on control plane nodes (master nodes).
     - Communicates with the API Server (API Server is the only component that directly interacts with etcd).
     - Uses gRPC API for communication.

#### scheduler
- it is responsible for assigning pods to nodes in a cluster. When a pod is created, it does not automatically get placed on a node. Instead, the scheduler evaluates available nodes and selects the most suitable one based on constraints and policies.
- The scheduler follows a two-phase process:
1. Filtering Phase:
     - The scheduler eliminates nodes that do not meet the pod’s requirements.
     - It considers factors like CPU, memory, taints, tolerations, node selectors, and affinity rules.
2. Scoring Phase:
     - After filtering, the scheduler assigns a score to each remaining node based on resource availability, topology, and other criteria.
     - The node with the highest score wins, and the pod is assigned to that node.
- Once the node is selected, the Kubernetes API server updates the pod spec with the node name, and the kubelet on that node takes over pod execution.

- Key Components of Scheduling
     - NodeSelector: Allows pods to specify which nodes they should run on.
     - Taints & Tolerations: Controls which pods can be scheduled on a node by marking nodes with specific conditions.
     - Affinity & Anti-affinity: Defines rules about pod placement, such as running pods on the same or different nodes.
     - Priority & Preemption: Ensures high-priority pods get scheduled first, evicting lower-priority pods if necessary.
     - Pod Overcommitment: Controls scheduling when requested CPU/memory is higher than available.

#### Controller Manager
- kube-controller-manager
     - it is a component that runs controller processes to ensure the cluster is in the desired state. It watches the Kubernetes API server for changes and makes adjustments to maintain the expected configuration
     - Example: If a pod crashes, the ReplicaSet controller detects the issue and creates a new pod to replace it.
 
- Key Responsibilities of the Controller Manager - it runs multiple controllers that manage different Kubernetes objects.
     - **Node Controller** -	Monitors the health of nodes and marks them as NotReady if unresponsive.
     - **Replication Controller** -	Ensures the desired number of pod replicas are running.
     - **Deployment Controller** -	Manages rolling updates and rollbacks for Deployments.
     - **DaemonSet Controller** -	Ensures DaemonSet pods run on every node.
     - **StatefulSet Controller** -	Manages stateful applications (e.g., databases) with stable pod identities.
     - **Job Controller** -	Runs batch jobs and ensures they complete successfully.
     - **CronJob Controller** -	Schedules and manages recurring jobs (like a Linux cron job).
     - **Service Account & Token Controller** -	Creates default service accounts and API tokens for authentication.
- Controller Manager Work?
     - The Controller Manager continuously watches the Kubernetes API Server for resource changes.
     - When a mismatch is detected (e.g., a pod crashes), the relevant controller takes action to restore the desired state.
     - Controllers use control loops that repeatedly check and update the cluster.
- Example Workflow - ReplicaSet Controller:
     - A ReplicaSet is defined with replicas: 3.
     - If one pod fails, the ReplicaSet controller detects the missing pod.
     - It creates a new pod to maintain the count of 3 replicas.

- cloud-controller-manager
     - The cloud-controller-manager is responsible for managing integration with cloud providers, such as AWS, GCP, or Azure.

#### Worker Node ( Data plane ) 
- is a machine (virtual or physical) in a Kubernetes cluster that runs application workloads (pods). Worker nodes form the Data Plane of Kubernetes, responsible for executing and managing containerized applications.
- Each worker node runs essential components to communicate with the Control Plane (Master Nodes) and execute scheduled workloads.
- Key Components
     - **Kubelet** -	Manages pod lifecycle on the node and communicates with the API Server.
     - **Container Runtime** - 	Runs containers inside pods (Docker, containerd, CRI-O).
     - **kube-proxy** -	Manages networking and load balancing for services.
     - **CNI Plugin (Container Network Interface)** -	Handles pod-to-pod networking across nodes.
- Worker Node Processes
     - **Pod Scheduling:** - The Scheduler in the Control Plane selects a worker node based on CPU, memory, and affinity rules.
     - **Pod Execution:** - The node downloads the container image and runs the pod using the container runtime.
     - **Pod Communication:** - The node's network stack enables pod-to-pod and pod-to-service communication.
- Worker Node Responsibilities
1. Run and Manage Pods:
     - The node hosts pods, which contain containers running applications.
     - Kubelet ensures that the right number of pods are running.
2. Handle Networking and Load Balancing:
     - kube-proxy enables communication between pods and services.
     - It applies iptables rules or IPVS to manage network traffic.
3. Communicate with the Control Plane:
     - The Kubelet constantly communicates with the API Server to get scheduling updates and send node health reports.
4. Monitor Node and Pod Health:
     - The Kubelet ensures pods stay healthy and restarts them if they fail.
     - The Node Controller in the Control Plane marks unhealthy nodes as NotReady.

-worker node components
     - API server
     - Etcd
     - Kube-scheduler
     - Kube-controller-manager
     - Cloud-controller-manager
     - Kubelet
     - kube-proxy
     - Container runtime
 
####  **key components**
- The key components of Kubernetes are the
     - Master Node
     - Worker Nodes
     - Pods
     - Services
     - Deployments
     - ReplicaSets
     - StatefulSets

#### kube-proxy
- its a networking component of Kubernetes that runs on each worker node and manages network connectivity for pods and services. It ensures that network traffic reaches the correct pod based on Kubernetes Services.
- Functions:
     - Implements Service IP-based load balancing (ClusterIP, NodePort, LoadBalancer).
     - Uses iptables or IPVS to route traffic efficiently.
     - Handles Pod-to-Pod and Pod-to-Service communication.
     - Works with Container Network Interface (CNI) plugins to manage networking.
- kube-proxy Works
     - When a Service is created, kube-proxy watches the Kubernetes API for updates
     - It modifies iptables rules or configures IPVS to ensure requests reach the correct pods.
     - Routes internal (**Pod-to-Pod**) and external (**Service-to-Pod**) traffic efficiently.
-  kube-proxy Modes (Networking Implementation) - supports three networking modes
     -  **iptables (default)** -	Uses iptables NAT rules to route traffic to pods. Efficient for small/medium clusters.
     -  **IPVS (Recommended for Large Clusters)** -	Uses Linux IP Virtual Server (IPVS) for high-performance load balancing.
     -  **Userspace (Legacy, Not Recommended)** -	Uses user-space proxying; slow and inefficient.

namespace used in Kubernetes?
     -  
43. What is a Kubernetes service?
     -  A Kubernetes service is an abstraction layer that exposes a set of pods as a network service, allowing them to communicate with each other and with other services outside the cluster.
44. Explain Kubernetes DNS.
     -  Kubernetes DNS is a service that provides DNS resolution for services and pods in a Kubernetes cluster, enabling them to discover and communicate with each other using DNS names.
45. What is a pod network in Kubernetes?
     -  A pod network is a network overlay that connects pods in a Kubernetes cluster, enabling them to communicate with each other across diff erent nodes.
46. Defi ne the Kubernetes CNI (Container Networking Interface).
     -  The Kubernetes CNI is a specifi cation that defi nes a standardized interface for integrating with container networking plugins, enabling diff erent networking solutions to work with Kubernetes clusters.

#### Rolling update
- info
     -  A rolling update is a strategy in Kubernetes that allows you to update a deployment by gradually replacing the existing pods with new ones.
     -  This ensures that the application remains available during the update process and reduces the risk of downtime.
- info 2
     - Rolling updates can be performed by updating the container image or configuration of a Deployment.
     - Kubernetes will gradually replace the old Pods with the new ones, minimizing downtime.
- info 2
     - Rolling updates in Kubernetes refer to the process of updating a deployment or a statefulset by gradually replacing old pods with new ones.
     - It ensures that the application remains available during the update process and avoids downtime.
     - Rolling updates follow a controlled strategy, gradually increasing the number of new pods while reducing the old ones, ensuring a smooth transition without impacting the overall availability of the application.

#### Rolling update stuck or not progressing
- Troubleshooting:
     - Check rollout status using kubectl rollout status.
     - Verify image versions in the deployment.
- Example Commands:
     - kubectl rollout status deployment <deployment-name>
     - kubectl set image deployment/<deployment-name> <container- name>=<new-image>

50. Describe the concept of horizontal pod autoscaling (HPA) in Kubernetes.
     -  Horizontal pod autoscaling (HPA) is a feature in Kubernetes that automatically scales the number of replica pods in a deployment based on CPU utilization or custom metrics. It allows the application to adapt to changing load conditions and ensures effi cient resource utilization.
51. What is a statefulset in Kubernetes?
     -  A statefulset is a workload API object in Kubernetes that is used for managing stateful applications. It provides guarantees for stable network identities and ordered, graceful deployment and scaling of pods. Statefulsets are typically used for applications that require stable network addresses or persistent storage.
52. Explain the concept of a secret in Kubernetes.
     -  A secret in Kubernetes is an API object that is used to store sensitive information, such as passwords, API keys, or TLS certifi cates. Secrets are stored securely within the cluster and can be mounted into pods as fi les or exposed as environment variables.
53. What is a persistent volume in Kubernetes?
     -  A persistent volume (PV) in Kubernetes is a storage abstraction that provides a way to store data independently of the pod's lifecycle. It allows data to
persist even when pods are terminated or rescheduled. Persistent volumes are used to provide storage for stateful applications.
54. Describe the role of a persistent volume claim (PVC) in Kubernetes.
     -  A persistent volume claim (PVC) is a request for storage by a user or a pod in Kubernetes. It is used to dynamically provision and bind a persistent volume to a pod. PVCs provide a way for users to request the type, size, and access mode of storage they need for their applications.

57. Describe the diff erence between a deployment and a statefulset in Kubernetes.
     -  The main diff erence between a deployment and a statefulset in Kubernetes lies in their use cases and the guarantees they provide. A deployment is primarily used for stateless applications and off ers easy scaling, rolling updates, and rollbacks. It manages a set of replica pods with no strict identity or reliance on stable network addresses.
On the other hand, a statefulset is designed for stateful applications that require stable network identities and ordered deployment and scaling. Statefulsets assign unique network identities and persistent storage to each pod, allowing them to maintain their identity and state even if they are rescheduled or restarted.
58. What is a pod disruption budget (PDB) in Kubernetes?
     -  A pod disruption budget (PDB) is a resource policy in Kubernetes that defi nes the maximum disruption that can be caused to a set of pods during a voluntary disruption event, such as a rolling update or a node eviction. It ensures
that a certain number of pods are always available and prevents excessive downtime or instability during updates or node failures.

60. What is the role of a namespace in Kubernetes?
     -  A namespace in Kubernetes provides a way to organize and isolate resources within a cluster. It allows diff erent teams or applications to have their own virtual clusters within a physical cluster. Namespaces help in avoiding naming confl icts, applying resource quotas, and segregating access control and network policies.
61. Describe the purpose of a label in Kubernetes.
     -  A label in Kubernetes is a key-value pair that can be attached to objects such as pods, services, or deployments. Labels are used to identify and select subsets of objects for various purposes. They enable grouping, fi ltering, and organizing resources, and they play a crucial role in defi ning selectors for services, deployments, and other Kubernetes components.
62. What is the role of a container registry in Kubernetes?
     -  A container registry in Kubernetes is a centralized repository for storing and distributing container images. It allows you to push and pull container images to and from the registry, making them available for deployment in Kubernetes clusters. Container registries facilitate versioning, distribution, and management of container images across multiple nodes and environments.
63. Explain the concept of a pod anti-affi nity in Kubernetes.
     -  Pod anti-affi nity in Kubernetes is a mechanism that allows you to defi ne rules for scheduling pods such that they are not co-located on the same node or with pods that have specifi c labels. It helps in distributing pods across diff erent nodes, enhancing fault tolerance, and improving availability by reducing the impact of node failures.
64. What is the role of the Kubernetes control plane?
     -  The Kubernetes control plane is a collection of components that manage and control the Kubernetes cluster. It includes the API server, scheduler, controller manager, and etcd, which is a distributed key-value store. The control plane is responsible for accepting and processing API requests, scheduling pods, maintaining desired state, and handling cluster-wide coordination and management tasks.

#### Describe the process of scaling a deployment in Kubernetes.
Scaling a deployment in Kubernetes involves adjusting the number of replica pods to meet the desired resource demands or application requirements. 
     - It can be achieved manually by updating the replica count in the deployment's specification, or automatically using horizontal pod autoscaling (HPA) based on CPU utilization or custom metrics. 
     - Scaling allows applications to handle increased load or improve resource utilization during low-demand periods.
     
#### readiness probe
A readiness probe is used to determine if a Pod is ready to receive traffic. Kubernetes uses this probe to determine when a Pod is fully operational and should be included in load balancing.
- A readiness probe in Kubernetes is a mechanism used to determine if a pod is ready to serve traffic. It periodically checks the health of a pod and reports its readiness status to the Kubernetes control plane. Readiness probes are essential for ensuring that only fully functional pods receive network traffic. If a pod fails the readiness probe, it is temporarily removed from the service's load balancer until it becomes ready again.
- A Readiness Probe in Kubernetes determines if a pod is ready to receive traffic. If the probe fails, Kubernetes removes the pod from the Service's endpoints, ensuring that no traffic is routed to an unhealthy pod.
- Use Readiness Probes
     - Prevents sending traffic to a pod before it's fully initialized.
     - Ensures rolling updates do not disrupt application availability.
     - Helps in graceful startup by delaying traffic until dependencies are available
- Use Cases
     - Database-backed applications (e.g., ensure DB connection before handling requests).
     - Applications with external dependencies (e.g., microservices that require another service to be running).
     - Long startup applications (e.g., Java applications with long initialization times)

#### Liveness Probe
A Liveness Probe in Kubernetes is used to check whether a container is still running. If the liveness probe fails, Kubernetes will restart the container automatically. This helps to recover from application failures where the container is alive but stuck in an unrecoverable state (e.g., deadlocks, unresponsive apps).
     - Liveness Probe Needed
     - Detects if an application is stuck or unresponsive.
     - Automatically restarts the container to recover from failures.
     - Ensures high availability and self-healing.
- it works:
     - Kubernetes sends an HTTP request to /health on port 8080.
     - If it returns a 2xx or 3xx status code, the container is healthy.
     - If it fails, Kubernetes restarts the container
- Use Liveness Probes
     - If your application occasionally hangs or deadlocks, use a liveness probe to restart it.
     - If your service should not be restarted unnecessarily, use a readiness probe instead.

#### readiness probe & liveness probe
- Info
     - Readiness probe: To prevent traffic to unready containers
          - Readiness Probe checks if a container is ready to serve traffic.
     - Liveness probe: To auto-recover from stuck or dead containers
          - Liveness Probe checks if a container is still running and healthy.
- Using both probes helps ensure zero-downtime deployments and auto-healing
     - readiness avoids sending traffic to unready pods
     - liveness ensures stuck containers are automatically restarted.
- Use Case:
     - Readiness: Wait until your app has fully loaded config or warmed up before getting traffic.
     - Liveness: Restart your app if it crashes or hangs.

#### configMap
- A ConfigMap is an API object in Kubernetes that allows you to store configuration data as key-value pairs separately from application code. This helps keep applications portable and environment-agnostic by injecting configuration settings at runtime.
- Why Use ConfigMaps
     - Decouples configuration from application code.
     - Allows dynamic updates without rebuilding containers.
     - Supports various data formats (YAML, JSON, environment variables, etc.).
     - Works seamlessly with Pods to pass configurations as environment variables, command-line arguments, or mounted volumes.
     - Centralized configuration management across multiple services.
- How Does a ConfigMap Work
     - Define a ConfigMap (storing configuration data).
     - Attach it to a Pod using environment variables or a mounted volume.
     - Pod reads configuration at runtime without modifying the container image.
     - Update the ConfigMap, and if needed, restart the Pod to apply changes.
- Best Practices for ConfigMaps
     - Use ConfigMaps for non-sensitive data (for secrets, use Kubernetes Secrets).
     - Ensure ConfigMaps are referenced in Deployments to reload updates.
     - Mount as a volume for large config files (e.g., JSON, YAML).
     - Use ConfigMap validation tools before applying changes.
- Conclusion
     - ConfigMaps help manage application configurations dynamically.
     - They keep config separate from code, making applications portable.
     - ConfigMaps can be used as environment variables or mounted as files.
     - They are best for non-sensitive configurations (use Secrets for credentials).

#### secret.
- its an API object that stores sensitive data such as passwords, API keys, TLS certificates, and authentication tokens. Unlike ConfigMaps, Secrets store data in Base64-encoded format for security purposes.
- Why Use Secrets?
     - Prevents hardcoding credentials inside container images or application code.
     - Restricts access to sensitive data using RBAC policies.
     - Supports encryption at rest (if enabled in Kubernetes).
     - Works with Pods, Volumes, and Environment Variables for secure data injection.
- How Does a Secret Work
     - Create a Secret (store sensitive data in a secure format).
     - Attach the Secret to a Pod as environment variables or mounted volumes.
     - Pod reads the Secret at runtime without exposing it in logs.
     - Control access using Kubernetes RBAC.
- Best Practices for Secrets
     - Enable encryption at rest for Secrets (--encryption-provider-config).
     - Use RBAC to limit access to Secrets (kubectl auth can-i get secret).
     - Do not store Secrets in Git (use external secret managers like AWS Secrets Manager, HashiCorp Vault).
     - Use Kubernetes service accounts instead of storing API keys in Secrets.
     - Rotate Secrets periodically to enhance security.
- Conclusion
     - Kubernetes Secrets manage sensitive data securely.
     - They prevent hardcoded credentials and support secure injection into Pods.
     - Use RBAC and encryption to protect Secrets from unauthorized access.
     - Rotate and manage Secrets with external tools for best security practices.


#### Describe the concept of a stateful application in Kubernetes.
-  A stateful application in Kubernetes refers to an application that requires stable network identities and persistent storage. Stateful applications typically maintain and rely on data or state that needs to be preserved across pod restarts or rescheduling. Examples include databases, key-value stores, and distributed systems. Stateful applications are often deployed using StatefulSets, which ensure ordered deployment, scaling, and management of the application's stateful pods.

#### What is the role of an Ingress in Kubernetes
-  An Ingress in Kubernetes is an API object that provides external access to services within a cluster. It acts as a centralized entry point for HTTP and HTTPS traffi c and allows for fl exible routing, SSL termination, and load balancing. By defi ning rules and paths, an Ingress controller can route incoming requests to the
appropriate services, enabling external access to applications running in the cluster.

#### Explain the concept of pod affi nity in Kubernetes.
- Pod affi nity in Kubernetes is a mechanism that allows you to defi ne rules for scheduling pods such that they are co-located on the same node or with pods that have specifi c labels. Pod affi nity is useful in scenarios where pods benefi t from being colocated, such as improving performance, reducing network latency, or optimizing resource utilization. It helps ensure that related pods are scheduled close to each other to enhance application performance or meet specifi c deployment requirements.

#### What are Kubernetes Operators
Kubernetes Operators are a way to package, deploy, and manage applications on Kubernetes using custom controllers. They extend the functionality of Kubernetes by automating complex application management tasks, such as provisioning, scaling, and upgrading. Operators are typically implemented using custom resources and controllers, allowing operators to defi ne and manage the lifecycle of specifi c applications or services in a declarative manner. They enable the automation of operational tasks and improve the overall manageability of applications in Kubernetes.

#### HorizontalPodAutoscaler- hpa
A HorizontalPodAutoscaler (HPA) in Kubernetes is a resource that automatically scales the number of pods in a deployment, replica set, or statefulset based on observed CPU utilization or custom metrics. The HPA controller continuously monitors the metrics of the targeted pods and adjusts the replica count to meet the defi ned resource utilization targets. This allows applications to automatically scale up or down based on demand, ensuring effi cient resource utilization and maintaining desired performance levels.

#### What is the purpose of a StatefulSet
A StatefulSet in Kubernetes is a workload controller used for managing stateful applications. Unlike deployments or replica sets, StatefulSets provide stable network identities and ordered deployment and scaling of pods. Each pod in a StatefulSet receives a unique and stable hostname, allowing stateful applications to maintain consistent network identities and configurations. StatefulSets are commonly used for deploying databases, distributed systems, or applications that require persistent storage and ordered scaling and management.

#### namespace
A namespace in Kubernetes is a virtual cluster that provides a way to divide and organize resources within a cluster. It acts as a logical boundary, allowing multiple users or teams to share a cluster without interfering with each other's resources. Namespaces provide isolation and resource allocation within a cluster and help in managing and securing applications by separating diff erent environments, such as development, staging, and production. They also enable better resource management, access control, and monitoring within a Kubernetes cluster.

#### service
A service in Kubernetes is an abstraction that provides a consistent way to access and communicate with pods running in a cluster. It acts as a stable endpoint for a set of pods, allowing other applications or services to access them without needing to know their specifi c IP addresses or individual locations. Services provide load balancing, service discovery, and internal network connectivity within the cluster, enabling scalable and resilient communication between diff erent components of an application.

#### label
A label in Kubernetes is a key-value pair that is attached to objects, such as pods, services, or deployments. Labels are used to identify and organize resources, allowing for fl exible grouping and selection. They can be used to categorize resources based on various attributes, such as environment, version, or purpose. Labels are instrumental in defi ning relationships between objects and are widely used for querying, selecting, and managing resources through Kubernetes operations and commands.

#### container runtime
A container runtime in Kubernetes is responsible for managing the execution and lifecycle of containers within a node. It provides the necessary infrastructure to run and manage containers, including container image management, resource isolation, and container lifecycle operations. Kubernetes supports multiple container runtimes, such as Docker, containerd, and CRI-O, allowing users to choose the runtime that best suits their requirements. The container runtime is an essential component of Kubernetes that enables the deployment and execution of containerized applications.
- A container runtime is responsible for starting and stopping containers on a node. Examples include Docker, containerd, and CRI-O.

#### Deployment
A Deployment in Kubernetes is a resource object that defi nes and manages a set of identical pods. It provides a declarative way to create and update pods, ensuring the desired number of replicas are running and handling rolling updates or rollbacks when changes are made.

#### Secret
A Secret in Kubernetes is a resource used to store and manage sensitive information, such as passwords, API keys, or certifi cates. Secrets are stored securely and can be mounted as fi les or injected as environment variables into pods, allowing applications to access the confi dential data.

#### Helm Chart
A Helm Chart in Kubernetes is a package format that contains all the necessary fi les, templates, and metadata to deploy a set of related Kubernetes resources. Helm is a package manager for Kubernetes, and using Helm Charts simplifi es the deployment and management of complex applications.

#### Ingress
An Ingress in Kubernetes is an API object that acts as an entry point to a cluster, allowing external traffi c to reach services within the cluster. It provides routing rules and load balancing capabilities, enabling the exposure of multiple services through a single external IP address.

#### Volume
A Volume in Kubernetes is a directory accessible to containers within a pod. It provides a way to store and share data between containers, as well as persist data beyond the lifetime of a pod. Volumes can be backed by diff erent
storage providers, such as local disk, network storage, or cloud-based storage systems.

#### Pod Security Policy
A Pod Security Policy in Kubernetes is a resource that defi nes a set of security conditions and restrictions for pods. It helps enforce security best practices by ensuring that pods adhere to certain security policies, such as restricting the use of privileged containers, enforcing container runtime constraints, or preventing host namespace sharing.

#### ResourceQuota
It is a resource object used to limit and manage the allocation of compute resources, such as CPU, memory, and storage, within a namespace. It allows administrators to defi ne usage limits, ensuring fair resource distribution and preventing resource exhaustion by applications within the namespace.

#### NetworkPolicy
- it is a resource object used to define and enforce network traffic rules and policies for pods. It provides fine-grained control over network access between pods, allowing administrators to specify ingress and egress rules based on IP addresses, ports, or other metadata.
- it is a security mechanism that controls inbound and outbound network traffic to and from Pods. It acts like a firewall for Pods, restricting or allowing communication between them based on defined rules.
- NetworkPolicies
     - Restrict unauthorized access between Pods.
     - Improve security by blocking unnecessary traffic.
     - Limit attack surface within a Kubernetes cluster.
     - Ensure compliance with security policies (e.g., PCI-DSS, GDPR).
- NetworkPolicy Work
     - By default, all Pods can communicate with each other in Kubernetes.
     - A NetworkPolicy restricts traffic to only allowed connections.
     - Policies apply only to Pods with specific labels and define:
          - Ingress rules (who can send traffic to the Pod).
          - Egress rules (where the Pod can send traffic).
     -  NetworkPolicies only work if a supported CNI plugin is used (e.g., Calico, Cilium, Weave, Antrea).
- Best Practices for NetworkPolicies
     - Use a "deny-all" default policy and explicitly allow needed traffic.
     - Apply NetworkPolicies at the namespace level for better security segmentation.
     - Monitor logs & alerts to detect unauthorized access attempts.
     - Combine NetworkPolicies with RBAC for enhanced security.
- Conclusion
     - NetworkPolicies enhance Kubernetes security by controlling traffic between Pods.
     - They prevent unauthorized communication and reduce attack surfaces.
     - They work only if a compatible CNI plugin is installed.
     - Use them with RBAC and monitoring tools for better security.

#### Helm Use Cases in a 3-Tier Architecture with Kubernetes
- A 3-tier architecture typically consists of:
     - Presentation Layer (Frontend - UI, Web Server)
     - Application Layer (Backend - APIs, Business Logic)
     - Database Layer (Database - MySQL, PostgreSQL, MongoDB)
- **Deploying a Scalable E-Commerce Application**
1. A company is deploying an e-commerce website using a 3-tier architecture on AWS EKS (Kubernetes). The goal is to automate deployments, manage configurations, and enable rollbacks efficiently.
- Architecture Breakdown
     - Frontend (React + Nginx) – Runs in a Kubernetes Deployment & exposed via an Ingress Controller.
     - Backend (Node.js API) – Microservices-based API handling business logic.
     - Database (PostgreSQL) – Stores user data, transactions, and product info.
2. Helm Chart for a 3-Tier Application
- Using Helm, we create a modular chart where each layer (frontend, backend, database) is a separate subchart, making it reusable and configurable.
- Helm Chart Structure
```ecommerce-helm-chart/
│── charts/                 # Subcharts for different tiers
│    ├── frontend/          # Frontend (React + Nginx)
│    ├── backend/           # Backend (Node.js API)
│    ├── database/          # Database (PostgreSQL)
│── templates/              # Parent chart templates
│── values.yaml             # Global configuration
│── Chart.yaml              # Metadata
```
1. Deploy Frontend (Presentation Layer)
     - The frontend runs in a Kubernetes Deployment with an Nginx web server and is exposed using an Ingress Controller.
     - Helm values file for frontend (frontend/values.yaml):
2. Deploy Backend (Application Layer)
     - The backend API (Node.js) interacts with the database and is exposed as a Kubernetes service for the frontend to communicate.
     - Helm values file for backend (backend/values.yaml):
3. Deploy Database (Data Layer)
     - The PostgreSQL database is deployed using a pre-built Helm chart (Bitnami’s PostgreSQL chart).
     - Helm values file for database (database/values.yaml):
4. Helm Chart Customization for Different Environments
     - Using Helm values overrides, we can deploy the same application in different environments with minimal changes.
     - Deploying to Staging vs. Production - staging-values.yaml
6. Helm Upgrade and Rollback for Zero-Downtime Deployments
     - If a new version of the backend needs to be deployed, Helm ensures a safe upgrade.
7. GitOps: Automating 3-Tier Deployment with ArgoCD
     - In a GitOps workflow, Helm can be integrated with ArgoCD for declarative deployment management.
8. Monitoring & Logging for the 3-Tier Application
     - Helm deploys Prometheus & Grafana for monitoring and ELK Stack (Elasticsearch, Logstash, Kibana) for logging.

#### K8s Issues - kubernetes

|Issue|Command to Use|Solution|
|-|-|-|
|Pod CrashLoopBackOff|kubectl describe pod <pod>|Check kubectl logs for errors. Fix application crash.|
|Pod ImagePullBackOff|kubectl get pods -o wide|Verify Docker image name and credentials.|
|Pod Pending|kubectl get events|Check for node capacity issues.|
|Service Not Accessible|kubectl describe svc <service>|Verify labels, selectors, and ports.|
|DNS Resolution Failure|nslookup my-service|Check CoreDNS logs and ensure correct service names.|
|OOMKilled (Out of Memory)|kubectl describe pod <pod>|Increase memory limits or optimize app memory usage.|

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

#### OpenShift.
OpenShift is a Kubernetes-based container orchestration platform developed by Red Hat. It provides developer and operational tools on top of Kubernetes for enterprise use.
     - Built-in CI/CD pipelines (via Tekton).
     - Secure by default: stricter security policies than Kubernetes.
     - Includes a web-based console for easier management.
     - Offers multi-tenancy, integrated monitoring, and RBAC.
- **Use Case Example**:
     - An enterprise looking to standardize on containers with enhanced security, compliance, and developer tooling may choose OpenShift over raw Kubernetes.

#### Autoscaling
Autoscaling in Kubernetes enables your application to scale automatically based on demand.
     - Horizontal Pod Autoscaler (HPA): Scales pod replicas based on CPU/memory/custom metrics.
     - Vertical Pod Autoscaler (VPA): Adjusts resource requests/limits of pods.
     - Cluster Autoscaler: Adds/removes nodes in the cluster based on pod requirements.
- Example: A web application pod scales from 2 to 5 replicas during peak hours using HPA when CPU utilization exceeds 70%.
     - kubectl autoscale deployment webapp --cpu-percent=70 --min=2 --max=5

#### pods are ephemeral
- Ephemeral means temporary or short-lived.
     - Pods in Kubernetes are not durable. If a pod dies (e.g., node crash, pod eviction), Kubernetes may create a new pod to replace it—but the new pod has a different IP and identity.
     - Any data stored in the pod's local storage is lost unless persistent storage (like a PVC) is used.
- Example: A frontend pod crashes and restarts — the new pod will have a different IP and be treated as a new instance.


- What happens after you edit a deployment and change the image
- What happens when you delete a deployment
- What is a Service in Kubernetes
- What Service types are there
- What is the difference between an external and an internal service
- What would you use to route traffic from outside the Kubernetes cluster to services within a cluster
- What is Ingress Controller
- What are some use cases for using Ingress
- What is the format of a configuration file
- What special namespaces are there by default when creating a Kubernetes cluster
- What can you find in kube-system namespace
- What kube-public contains
- What kube-node-lease contains
- What default namespace contains
- What is Resource Quota
- What kubectl exec does
- What kubectl get all does
- What the command kubectl get pod does
- What kubectl apply -f [file] does
- What the command kubectl api-resources --namespaced=false does
- What kubectl logs [pod-name] command does
- What kubectl describe pod [pod name] does command does
- What kubectl get componentstatus does
- What is Minikube
- What the Kubernetes Scheduler does
- What happens to running pods if if you stop Kubelet on the worker nodes
- What happens what pods are using too much memory (more than its limit)
- What is the control loop How it works
- What is an Operator
- What components the Operator consists of
- What is the Operator Framework
- What components the Operator Framework consists of
- What openshift-operator-lifecycle-manager namespace includes
- What is kubconfig What do you use it for
- What is the purpose of ReplicaSet
- What happens when a replica dies
- What type: Opaque in a secret file means What other types are there
- What types of persistent volumes are there
- What is Reclaim Policy
- What reclaim policies are there
- What is RBAC
- What is the difference between Role and ClusterRole objects
- What Kube Proxy does
- What "Resources Quotas" are used for and how
- What does being cloud-native mean
- What QoS classes are there
- What is Kubeconfig
- What is Container Orchestration
- What are the features of Kubernetes
- What is Google Container Engine
- What is Heapster
- What is the syntax for the Kube-proxy command
- What is the syntax for the Kubectl command
- What is Kubelet
- What are the different components of Kubernetes Architecture
- What do you understand by Kube-proxy
- What is the Kubernetes controller manager
- What are the different types of controller manager running on the master node
- What do you understand by load balancer in Kubernetes
- What is Ingress network
- What do you understand by Cloud controller manager
- What are the different types of cloud controller manager
- What is a Headless Service
- What are federated clusters
- What is the difference between a daemonset, a deployment, and a replication controller
- What is a sidecar container, and what would you use it for
- What are K8s
- What is a node in Kubernetes
- What does the node status contain
- What process runs on Kubernetes Master Node
- What is the job of the kube-scheduler
- What is a cluster of containers in Kubernetes
- What is a Namespace in Kubernetes
- What are the different services within Kubernetes
- What is NodePort
- What is Kube-proxy
- What is the difference between config map and secret
- What are the tools that are used for container monitoring
- What are the important components of node status
- What are the labels in Kubernetes
- What is Sematext Docker Agent
- What is ContainerCreating pod
- What do you mean by Persistent Volume Claim
- What will happen while adding new API to Kubernetes
- What is kubectl drain
- What is a Swarm in Docker
- What is Kubernetes Log
- What are the types of Kubernetes Volume
- What is the Kubernetes Network Policy
- What is the Kubernetes
- What is Kubernetes and how to use it
- What is the meaning of Kubernetes
- What is a docker
- What is a cluster in Kubernetes
- What is Openshift
- What is a namespace in Kubernetes How do you create a Namespace
- What is Docker and what does it do
- What is a docker in cloud
- What is a cluster of containers
- What is the use of kube-controller-manager
- What are Kubernetes controllers
- What is kube-scheduler
- What are Expanders
- What are init containers
- What are Kubernetes Components Master component  Node component
- What are Kubernetes pods
- What are the API versions available Explain.
- What are the benefits of Kubernetes
- What are the functions of Replication controller
- What are the key best practices for running Cluster Autoscaler
- What are the kubectl commands you are aware of
- What are the parameters to CA
- What are the Service Level Objectives for Cluster Autoscaler
- What are the types of Kubernetes pods How do you create them
- What are the types of Kubernetes services
- What are the types of multi-container pod patterns (Explain each type with examples)
- What do you know about Kubernetes Service
- What do you know about Minions Explain.
- What do you know about Pods in Kubernetes
- What do you know about Selectors and what are the types of selectors in Kubernetes API
- What do you know about Sematext Docker Agent
- What is Recreate and Rolling Update in Deployment strategy
- What is application deployment in Kubernetes
- What is Kubernetes images
- What is Kubernetes Name space
- What is Node Controller
- What is Persistent Volume Claim
- What is Persistent Volume
- What is volumes What are the differences between Docker volumes and Kubernetes Volumes
- What events are emitted by CA
- What happens if  daemonset can be set to listen on a specific interface since the Anycast IP will be assigned to a network interface alias
- What happens in scale-up when I have no more quota in the cloud provider
- What happens when a master fails What happens when a worker fails
- What is a container cluster
- What is a DaemonSet
- What is a Heapster
- What is a PetSet or StatefulSet
- What is are the reasons why Kubernetes is more useful by walking back in time
- What is Autoscaling in Kubernetes.
- What is Cloud Controller manager
- What is Cluster  What is Cluster IP
- What is Cluster Autoscaler
- What is Container Resource monitoring
- What is Federated clusters
- What is Flannel & why we use it
- What is GKE
- What is Headless service
- What is IngressNetwork and IngressController
- What is KUBE proxy
- What is kubectl & kubelet
- What is kubectl How do you use it
- What is Kubernetes load balancing
- What is Kubernetes and why is it damn popular
- What is Kubernetes
- What is load balancing on Kubernetes
- What is node affinity and pod affinity
- What is Node or Minions
- What is Node port
- What is POD
- What is Replica
- What is Replica controller
- What is Replication controller & what it does
- What is Secrets in Kubernetes.
- What is the function of Kube-apiserver
- What is the future scope for Kubernetes
- What is the impact of upgrading kubelet if we leave the pods on the worker node - will it break running pods why
- What is the ingress, is it something that runs as a pod or on a pod
- What is the use of the API server in Kubernetes
- What kind of object do you create, when your dashboard like application, queries the Kubernetes API to get some data
- What monitoring and metrics tools do people use for Kubernetes
- What runs inside the kubernetes worker nodes
- What types of pods can prevent CA from removing a node
- What versions of docker are supported
- What you will do if one master got corrupted, can we create multiple masters
- What you will do in case any pod deleted
- What is Labels and Annotations in kubernetes
- What is Kubectl command.
- What is Minions
- What is Pods in Kubernetes Context
- What is Replication Controllers and Replication Sets in Kubernetes 
- What is Volumes and Persistent Volumes in kubernetes
- Name the initial namespaces from which Kubernetes starts
- Name the process which runs on Kubernetes Master Node
- Write a command to create and fetch the deployment.
- Write a command to check the status of deploymentand to update a deployment.
- Write commands to control the Namespace.
- Explain what is a pod
- Explain StatefulSet
- Explain Kubernetes Secrets
- Explain "Persistent Volumes". Why do we need it
- Explain Storage Classes
- Explain "Dynamic Provisioning" and "Static Provisioning"
- Explain Access Modes
- Explain the Role and RoleBinding" objects
- Explain what Kubernetes Service Discovery means
- Explain "Horizontal Pod Autoscaler"
- Explain the pet and cattle approach of infrastructure with respect to kubernetes
- Explain Labels. What are they and why would one use them
- Explain Selectors
- Explain Prometheus in Kubernetes.
- Explain Replica set.
- Explain the types of Kubernetes pods.
- Explain Kubernetes architecture with a neat diagram
- Explain the Architecture Layers of Kubernetes
- Explain the Network Policy in Kubernetes.
- Why to use namespaces What is the problem with using one default namespace
- Why to create kind deployment, if pods can be launched with replicaset
- Why do we need Operators
- Why uses Kube-apiserver
- Why do we use Docker
- Why are some of my pods in an Unknown state
- Why do I see 504 errors from my Ingress during deploys
- Why do we need Kubernetes and what it can do
- Why Docker isnt enough Why do we need Kubernetes
- Why is my terminal screwed up
- Why should I use Kubernetes
- Why we using kops
- Why you are using kubectl can you explin why we are using 
- Which command you run to view your nodes
- Which command you run to view all pods running on all namespaces
- Which parts a configuration file has
- Which resources are accessible from different namespaces
- Which service and in which namespace the following file is referencing
- Which components can't be created within a namespace
- Which process runs on Kubernetes master node
- Which process runs on Kubernetes non-master node
- Which process validates and configures data for the api objects like pods, services
- Which container runtimes supported by Kubernetes
- Which version on Cluster Autoscaler should I use in my cluster
- How many containers can a pod contain
- How to delete a pod
- How to create a deployment
- How to edit a deployment
- How to delete a deployment
- How make an app accessible on private or external network
- How to get information on a certain service
- How to verify that a certain service forwards the requests to a pod
- How to list Ingress in your namespace
- How to configure a default backend
- How to configure TLS with Ingress
- How to get latest configuration of a deployment
- How to list all namespaces
- How to get the name of the current namespace
- How to create a namespace
- How to switch to another namespace In other words how to change active namespace
- How to create a Resource Quota
- How to list all the components that bound to a namespace
- How to create components in a namespace
- How to see all the components of a certain application
- How to print information on a specific pod
- How to execute the command "ls" in an existing pod
- How to create a service that exposes a deployment
- How to create a pod and a service with one command
- How to scale a deployment to 8 replicas
- How to get list of resources which are not in a namespace
- How to delete all pods whose status is not "Running"
- How to display the resources usages of pods
- How do you monitor your Kubernetes
- How Operator works
- How a ReplicaSet works
- How to create a Secret from a key and value
- How to create a Secret from a file
- How to create a Secret from a configuration file
- How to use ConfigMaps
- How to delete a pod instantly
- How would you troubleshoot your cluster if some applications are not reachable any more
- How does scheduling work in kubernetes
- How are labels and selectors used
- How is Kubernetes related to Docker
- How does Kubernetes simplify containerized Deployment
- How can you separate resources
- How can you get a static IP for a Kubernetes load balancer
- How do you make changes in the API

#### setup Kubernetes.
- How 2 containers inside a pod communicate with each other
- How can I check what is going on in CA 
- How can I configure overprovisioning with Cluster Autoscaler
- How can I get the host IP address from inside a pod
- How can I monitor Cluster Autoscaler
- How can I prevent Cluster Autoscaler from scaling down a particular node
- How can I run e2e tests
- How can I scale a node group to 0
- How can I scale my cluster to just 1 node
- How can I update CA dependencies (particularly k8s.io/kubernetes)
- How can you rollbck the previous version of application in Kuberntes
- How do CPU and memory requests and limits work
- How do I access the Kubenernetes API from within a pod
- How do I build a High Availability (HA) cluster
- How do I configure credentials to download images from a private docker registry
- How do I debug a ContainerCreating pod
- How do I debug a CrashLoopBackoff pod
- How do I debug a Pending pod
- How do I determine the status of a Deployment
- How do I expose a Service to a host outside the cluster
- How do I force a pod to run on a specific node
- How do I force replicas of a pod to split across different nodes
- How do I get all the pods on a node
- How do I give individual pods DNS names
- How do I put variables into my pods
- How do I rollback a Deployment
- How do I rollback the Deployment
- How do I update all my pods if the image changed but the tag is the same
- How do you build a High Availability (HA) cluster
- How do you create an application in Kubernetes
- How do you create secrets in Kubernetes
- How do you create users
- How do you deploy a feature with zero downtime in Kubernetes
- How do you drain the traffic from a Pod during maintenance
- How do you initiate a rollback for an application
- How do you monitor your Kuberenets
- How do you package Kubernetes applications
- How do you test a manifest without actually executing it
- How do you tie service to a pod or to a set of pods
- How do you update, delete and rollback in a Deployment strategy
- How does a Kubernetes Service work
- How does CA deal with unready nodes
- How does Cluster Autoscaler remove nodes
- How does Cluster Autoscaler work with Pod Priority and Preemption
- How does container know that application is getting failure
- How does DNS work in Kubernetes
- How does Horizontal Pod Autoscaler work with Cluster Autoscaler
- How does Kubernetes relate to Docker
- How does scale-down work
- How does scale-up work
- How does that deployment happens into containers/POD automatically
- How does the Kubernetes Cluster work
- How fast is Cluster Autoscaler
- How fast is HPA when combined with CA
- How is Cluster Autoscaler different from CPU-usage-based node autoscalers
- How Kubernetes is related to docker
- How Kubernetes simplify the containerized application deployment process
- How many containers can be launched in a node
- How many containers can run in a pod
- How many nodes we required to create kubernetes cluster
- How many projects you used kubernetes
- How service that selects apps based on the label and has an externalIP
- How should I test my code before submitting PR
- How should you connect an app pod with a database pod
- How the 2 pods communicate with each other
- How to attach a volume in cluster at some time the container will be deleted then rs will re-create new container then how to attach that container automatically and how to restore the volume automatically to re-created container
- How to configure a default ImagePullSecret for any deployment
- How to forward port `8080 (container) -> 8080 (service) -> 8080 (ingress) -> 80 (browser)` how is it done
- How to Install Kubernetes
- How to make quorum of cluster
- How to monitor that a Pod is always running
- How to set a static IP for Kubernetes load balancer
- How to set PDBs to enable CA to move kube-system pods
- How will you do monitoring in Kubernetes
- How you will link when the docker containers is in different virtual machine is there any configuration in docker compose file are any command or any variable
- When you delete a pod, is it deleted instantly (a moment after running the command)
- When do I use Kubernetes
- When you want to monitor the health and performance of multiple containers.
- When does Cluster Autoscaler change the size of a cluster
- Where Kubernetes gets the status data (which is added to the configuration file) from
- Where Kubernetes cluster data is stored
- Where can I find the designs of the upcoming features
- Where is the Kubernetes cluster data stored
- Difference -  Pod and a Job Differentiate the answers as with examples)
- Difference -  config map and secret (Differentiate the answers as with examples)
- Difference -  docker cloud and docker swarm
- Difference -  externalIP and loadBalancerIP 
- Difference -  kubctl & kops
- Difference -  kubectl and minikube
- Difference -  Kubernetes and Docker Swarm
- Difference -  replication controllers and replica sets
- Difference -  Flannel & Calico
- Difference -  nodeport, clusterIP, load balancer & ingress
- Difference -  rc and rs
- In  Kubernetes - A Pod is running 2 containers, when One container stops - another Container is still running, on this event, I want to terminate this Pod
- Do rolling updates declared with a deployment take effect if I manually delete pods of the replica set with kubectl delete pods or with the dashboard Will the minimum required a number of pods be maintained
- Are all of the mentioned heuristics and timings final
- Are deployments with more than one replica automatically doing rolling updates when a new deployment config is applied
- Basic Usage Questions
- CA doesnt work, but it used to work yesterday. Why
- Can you use a Deployment for stateful applications
- Can we use many claims out of a persistent volume
- Can I isolate namespaces from each other
- Can I use variables or otherwise parameterize my yaml deployment files
- Can pods mount NFS volumes
- Can we use many claims out of a persistent volume Explain
- Can you tell me some commands using in kubernetes 
- Can you tell me the command for creating kubernetes cluster in vm
- Deploy a pod called "my-pod" using the nginx:alpine image
- Describe in detail what the following command does kubectl create deployment kubernetes-httpd --image=httpd
- Describe how roll-back works
- Describe in detail what is the Operator Lifecycle Manager
- Describe how you one proceeds to run a containerised web app in K8s, which should be reachable from a public URL.
- Describe what CustomResourceDefinitions there are in the Kubernetes world What they can be used for
- Describe in brief the working of the master node in Kubernetes
- Describe the architecture of Kuberenets
- Describe the history of Kubrnetes
- Different Ways to provide API-Security on Kubernetes
- Does CA respect GracefulTermination in scale-down
- Does CA respect node affinity when selecting node groups to scale up
- Does CA work with PodDisruptionBudget in scale-down
- Does the container restart When applying/updating the secret object (kubectl apply -f mysecret.yml)  If not, how is the new password applied to the database
- Does the rolling update with state full set replicas =1 makes sense
- For advanced networking between containers hosted across the cluster.
- Having a Pod with two containers, can I ping each other like using the container name
- I have a configmap for 3 files that are going to be mounted in supposing "fluentd/etc/" and the respective files would be fluent.conf,  kubernetes.conf, systemd.conf, config map in deployment.yaml is like this
- I have a couple of nodes with low utilization, but they are not scaled down. Why
- I have a couple of pending pods, but there was no scale-up
- I have one POD and inside 2 containers are running one is Nginx and another one is  wordpress So, how can access these 2 containers from the Browser with IP address
- If a node is tainted, is there a way to still schedule the pods to that node
- If a pod exceeds its memory "limit" what signal is sent to the process
- If any container down in my cluster how you will rectify
- If I have multiple containers running inside a pod, and I want to wait for a specific container to start before starting another one.
- If installed kubernetes how you will deploy this containers into kubernetes cluster
- If you have a pod that is using a ConfigMap which you updated, and you want the container to be updated with those changes, what should you do
- If you have multiple containers in a Deployment file, does use the HorizontalPodAutoscaler scale all of the containers
- I'm running cluster with nodes in multiple zones for HA purposes. Is that supported by Cluster Autoscaler
- Is Cluster Autoscaler an Alpha, Beta or GA product
- Is Cluster Autoscaler compatible with CPU-usage-based node autoscalers
- Is it possible to route traffic from outside the Kubernetes cluster directly to pods
- Is it possible to run docker inside a pod
- Is there a way to make a pod to automatically come up when the host restarts
- Is there any other way to update configmap for deployment without pod restarts
- Kubernetes objects made up of what
- Lets say a Kubernetes job should finish in 40 seconds, however on a rare occasion it takes 5 minutes, How can I make sure to stop the application if it exceeds more than 40 seconds
- Let's say you have three namespaces: x, y and z. In x namespace you have a ConfigMap referencing service in z namespace. Can you reference the ConfigMap in x namespace from y namespace
- List tools for container orchestration.
- List out Kubernetes Objects and Workloads
- List the Kubernetes volume you are aware of.
- Mention the namespaces that initially the Kubernetes starts with
- N number of docker containers deployed to different vms how will you manage there is no kubernetes installed
- Once update is installed, add new key for GPG using the command:
- Our applications are decentralized I dont want distributed environment if any thing happens to the master all will collapse , can we create multiple masters
- Should I use a CPU-usage-based node autoscaler with Kubernetes
- Should I use Replication Controllers
- State the functions of Kubernetes name space.
- State the important features of Kubernetes.
- Suppose you have to use database with your application but well, if you make a database container-based deployment. how would the data persist
- Tell me about Google container Engine.
- Tell me about the functions of Kubernetes Jobs.
- Tell me the command to create cluster
- To deploy 1000s of containers in a single command.
- To detect fails/crashes of containers and fix them.
- To scale up and scale down the number of containers.
- To customize deployment of containers.
- To support all cloud service environments: many cloud providers offer built-in Kubernetes services.
- To upgrade all the containers in a single command.
- To roll back container updates if something goes wrong.
- To support a wide variety of authentication and authorization services.
- Using create command along with kubectl, what are the things possible
- We have nearly 15 nodes in my organization all are decentralized so which node I need to create as a master Is their any possibility to make all the machines as masters
- You suspect one of the pods is having issues, what do you do
- You have one Kubernetes cluster and multiple teams that would like to use it. You would like to limit the resources each team consumes in the cluster. Which Kubernetes concept would you use for that


#### Unable to connect to the cluster
- Troubleshooting:
     - Check kubeconfig file for correct cluster information.
     - Verify network connectivity to the cluster.
- Example Commands:
     - kubectl config view kubectl cluster-info

#### Pod stuck in Pending state
- Troubleshooting:
     - Check events for the pod using kubectl describe pod.
     - Inspect the pod's YAML for resource constraints or affinity issues.
- Example Commands:
kubectl describe pod <pod-name>
kubectl get events --namespace <namespace>

#### Insufficient resources to schedule pod Troubleshooting:
     - Check resource requests and limits in the pod specification.
     - Verify node resources using kubectl describe node.
- Example Commands:
     - kubectl describe pod <pod-name> kubectl describe node <node-name>

#### ImagePullBackOff
- Troubleshooting:
     - Verify the image name and availability.
     - Check image pull credentials using kubectl describe pod.
- Example Commands:
     - kubectl describe pod <pod-name>
     - kubectl get pods --namespace <namespace> - o=jsonpath='{.items[*].status.containerStatuses[*].state}'

#### CrashLoopBackOff
- Troubleshooting:
     - Check container logs for details on the crash.
     - Inspect pod events using kubectl describe pod.
- Example Commands:
     - kubectl logs <pod-name> <container-name> kubectl describe pod <pod-name>

#### Unauthorized access
- Troubleshooting:
     - Verify RBAC permissions for the user.
     - Check kubeconfig for correct credentials.
- Example Commands:
     - kubectl auth can-i --list kubectl config view

#### ConfigMap not updating in the pod
- Troubleshooting:
     - Check if the ConfigMap is updated.
     - Verify that the pod is configured to use the latest version.
- Example Commands:
     - kubectl get configmap <configmap-name> -o yaml kubectl describe pod <pod-name>

#### Service not reachable
- Troubleshooting:
     - Check service endpoints using kubectl describe service.
     - Verify network policies and firewall rules.
- Example Commands:
     - kubectl describe service <service-name> kubectl get networkpolicies

#### Node not ready
- Troubleshooting:
     - Check node status with kubectl get nodes.
     - Review kubelet logs on the node for issues.
- Example Commands:
     - kubectl get nodes
     - kubectl describe node <node-name>

#### PersistentVolumeClaim (PVC) pending
- Troubleshooting:
     - Verify available storage in the cluster.
     - Check storage class and provisioner.
- Example Commands:
     - kubectl get pvc
     - kubectl describe storageclass

#### VolumeMounts not working in pod
- Troubleshooting:
     - Check pod's YAML for correct volume mounts.
     - Verify if the volume exists and is accessible.
     - Example Commands:
kubectl describe pod <pod-name> kubectl get pv

#### Pod Security Policies (PSP) blocking pod
- Troubleshooting:
     - Check PSP rules and RBAC for the pod.
     - Inspect pod events using kubectl describe pod.
- Example Commands:
     - kubectl get psp
     - kubectl describe pod <pod-name>

#### ServiceAccount permissions
- Troubleshooting:
     - Verify ServiceAccount permissions using kubectl auth can-i.
     - Check RBAC roles and role bindings.
- Example Commands:
     - kubectl auth can-i --list -- as=system:serviceaccount:<namespace>:<serviceaccount-name>
     - kubectl get roles,rolebindings --namespace <namespace>

#### NodeSelector not working
- Troubleshooting:
     - Check pod's YAML for correct node selector.
     - Verify that nodes have the required labels.
- Example Commands:
     - kubectl describe pod <pod-name> kubectl get nodes --show-labels

#### Ingress not routing traffic
- Troubleshooting:
     - Check Ingress resource for correct backend services.
     - Verify that the Ingress controller is running.
- Example Commands:
     - kubectl describe ingress <ingress-name>
     - kubectl get pods --namespace <ingress-controller-namespace>

#### Unable to scale deployment
- Troubleshooting:
     - Verify available resources in the cluster.
     - Check replica count in the deployment specification.
- Example Commands:
     - kubectl get deployments
     - kubectl describe deployment <deployment-name>

#### Custom Resource Definition (CRD) not creating resources
- Troubleshooting:
     - Check CRD definition for correct syntax.
     - Verify controller logs for errors.
- Example Commands:
     - kubectl get crd
     - kubectl describe crd <crd-name>

#### Pod in Terminating state
- Troubleshooting:
     - Check for stuck finalizers in pod metadata.
     - Force delete pod using kubectl delete pod --grace-period=0.
- Example Commands:
     - kubectl get pods --all-namespaces --field- selector=status.phase=Terminating
     - kubectl delete pod <pod-name> --grace-period=0 –force
 
#### Resource quota exceeded
- Troubleshooting:
     - Check resource quotas for the namespace.
     - Verify resource usage in the namespace.
- Example Commands:
     - kubectl describe quota --namespace <namespace> kubectl top pods --namespace <namespace>

#### Node draining or cordoning
- Troubleshooting:
     - Check node conditions and events.
     - Use kubectl drain with caution.
- Example Commands:
     - kubectl get nodes
     - kubectl describe node <node-name>
     - kubectl drain <node-name> --ignore-daemonsets

#### Resource creation timeout
- Troubleshooting:
     - Check for issues with the API server.
     - Verify network connectivity to the API server.
- Example Commands:
     - kubectl get events --sort-by='.metadata.creationTimestamp' kubectl describe pod <pod-name>
 
#### Pod stuck in ContainerCreating state
- Troubleshooting:
     - Check container runtime logs on the node.
     - Inspect kubelet logs for errors.
- Example Commands:
     - kubectl get pods
     - kubectl describe pod <pod-name>

#### Invalid YAML syntax
- Troubleshooting:
     - Validate YAML syntax using online tools or linters.
     - Check for indentation and formatting issues.
- Example Commands:
     - kubectl apply -f <file.yaml> --dry-run=client

#### etcd cluster issues
- Troubleshooting:
     - Check etcd logs for errors.
     - Verify etcd cluster health.
- Example Commands:
     - kubectl get events --all-namespaces --field- selector=involvedObject.kind=Pod,involvedObject.name=etcd
     - kubectl exec -it etcd-pod-name --namespace kube-system -- sh etcdctl member list
     - etcdctl cluster-health
