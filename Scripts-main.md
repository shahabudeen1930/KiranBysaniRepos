#### Bash Script – Monitor and Restart Service

```
#!/bin/bash

SERVICE="nginx"
if ! systemctl is-active --quiet $SERVICE; then
  echo "$SERVICE is down, restarting..."
  sudo systemctl restart $SERVICE
else
  echo "$SERVICE is running."
fi
```

- Explanation:
     - #!/bin/bash: Shebang tells the system to use bash shell to interpret the script.
     - SERVICE="nginx": We're setting a variable to represent the service to monitor.
     - Systemctl is-active: Checks if the service is running.
     - --quiet: Suppresses output, just returns status.
     - ! ...: The ! negates the check – so it triggers if service is NOT active.
     - systemctl restart: Restarts the service if it’s down.


#### Python Script – Get Pod Status via kubectl

```
import subprocess
import json
result = subprocess.run(["kubectl", "get", "pods", "-o", "json"], capture_output=True, text=True)
pods = json.loads(result.stdout)

for pod in pods["items"]:
    name = pod["metadata"]["name"]
    status = pod["status"]["phase"]
    print(f"Pod: {name} | Status: {status}")
```

- Explanation:
     - subprocess.run: Executes shell commands (like kubectl) from Python.
     - -o json: Gets structured pod info.
     - json.loads: Parses the JSON output.
     - Loops through pods to extract and display name and status.

#### Terraform – Launch EC2 Instance

```
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "web" {
  ami           = "ami-0c02fb55956c7d316" # Amazon Linux 2
  instance_type = "t2.micro"

  tags = {
    Name = "DevOps-Demo"
  }
}
```

- Explanation:
     - provider "aws": Specifies AWS as the cloud provider.
     - region: Defines the AWS region.
     - resource "aws_instance": Declares an EC2 instance.
     - ami: Specifies the Amazon Machine Image ID.
     - instance_type: Defines the size/type of the instance.
     - tags: Helps with resource identification.

#### Dockerfile – Python Flask App

```
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

- Explanation:
     - FROM: Base image with Python 3.9.
     - WORKDIR: Sets working directory inside the container.
     - COPY requirements.txt: Copies dependency file.
     - RUN pip install: Installs required packages.
     - COPY . .: Copies all files to container.
     - CMD: Starts the Flask app.

#### YAML – Deployment with Environment Variable - Kubernetes

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
          image: nginx:latest
          env:
            - name: ENVIRONMENT
              value: "production"
```

- Explanation:
     - Deployment: Ensures specified number of pods are running.
     - replicas: 3: Run 3 pods.
     - matchLabels: Matches pods to this deployment.
     - template: Pod definition inside deployment.
     - env: Sets an environment variable inside the container.
- Contents 
     - **apiVersion** and kind define the resource type
     - **metadata** specify the details of deployment such as the name of the deployment, labels.
     - **spec** defines the desired state of the Deployment.
     - **replicas** specify the desired number of pods to maintain.
     - **selector** - specifies on which pod the replica configuration should be applied with the help of the label of the pod.
     - **template** describes the pod template and how deployment will use it to create new pods.
     - **containers** will list the containers to run within the pod.
     - **name** specifies the name of the container
     - **image** specifies the name that will be used to run the container. The image will be a Docker Image.
     - **containerport** specifies the port at which the container will listen for incoming traffic.

#### Jenkinsfile – Basic Build Pipeline

```
pipeline {
  agent any

  stages {
    stage('Build') {
      steps {
        echo 'Building the project...'
      }
    }
    stage('Test') {
      steps {
        echo 'Running tests...'
      }
    }
    stage('Deploy') {
      steps {
        echo 'Deploying application...'
      }
    }
  }
}
```

- Explanation:
     - pipeline: Defines Jenkins pipeline in Declarative syntax.
     - agent any: Runs the pipeline on any available agent.
     - stages: Represents the CI/CD stages.
     - Each stage has steps, where commands run.

#### JSON – Prometheus Alert Rule

```
{
  "groups": [
    {
      "name": "cpu-alerts",
      "rules": [
        {
          "alert": "HighCPUUsage",
          "expr": "100 - (avg by(instance)(rate(node_cpu_seconds_total{mode=\"idle\"}[1m])) * 100) > 80",
          "for": "2m",
          "labels": {
            "severity": "warning"
          },
          "annotations": {
            "summary": "Instance {{ $labels.instance }} high CPU usage",
            "description": "CPU usage is above 80% for more than 2 minutes."
          }
        }
      ]
    }
  ]
}
```

- Explanation:
     - groups and rules: Prometheus alertmanager syntax.
     - expr: PromQL expression checking CPU idle time and calculating CPU usage.
     - for: "2m": Triggers if condition holds for 2 minutes.
     - labels: Categorizes severity.
     - annotations: Describes the alert message.

#### ARM Template – Deploy Azure Storage Account

```
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountName": {
      "type": "string"
    },
    "location": {
      "type": "string",
      "defaultValue": "eastus"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2021-04-01",
      "name": "[parameters('storageAccountName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "StorageV2",
      "properties": {}
    }
  ]
}
```

- Explanation :
     - Schema: Defines the version of the template format.
     - Parameters: Input values like storageAccountName and location.
     - Resources: Describes what to deploy – here, a Storage Account.
     - "kind": "StorageV2": Latest and most flexible storage account kind.
     - "sku": "Standard_LRS": Standard performance, locally redundant

####  Azure Function (Python) – HTTP Trigger

```
import logging
import azure.functions as func
def main(req: func.HttpRequest) -> func.HttpResponse:
    name = req.params.get('name')
    if not name:
        try:
            req_body = req.get_json()
        except ValueError:
            pass
        else:
            name = req_body.get('name')

    if name:
        return func.HttpResponse(f"Hello, {name}!")
    else:
        return func.HttpResponse(
            "Please pass a name on the query string or in the request body",
            status_code=400
        )
```

- Explanation:
     - HTTP trigger: Function runs when an HTTP request is received.
     - req.params.get('name'): Reads ?name= from query string.
     - req.get_json(): Fallback to reading JSON body.
     - Returns a dynamic response or 400 error.

#### PowerShell Script – Create Azure Resource Group and VM

```
# Variables
$location = "EastUS"
$resourceGroupName = "DevOpsDemoRG"
$vmName = "DemoVM"
# Create Resource Group
New-AzResourceGroup -Name $resourceGroupName -Location $location
# Create a Virtual Machine
New-AzVM `
  -ResourceGroupName $resourceGroupName `
  -Name $vmName `
  -Location $location `
  -Image "Win2019Datacenter" `
  -Credential (Get-Credential) `
  -Size "Standard_B1s"
```

- Explanation:
     - New-AzResourceGroup: Creates a new resource group.
     - New-AzVM: Deploys a new Windows VM.
     - Get-Credential: Prompts for username/password for the VM.
     - -Image: OS version.
     - -Size: VM type (cost-effective B-series).
  
#### Lambda Function (Python) – API Gateway Trigger

```
def lambda_handler(event, context):
    name = event.get('queryStringParameters', {}).get('name', 'World')
    return {
        'statusCode': 200,
        'body': f'Hello, {name}!'
    }
```

- Explanation:
     - event: Contains request data, like headers, query params, etc.
     - context: Metadata about the Lambda execution (timeout, memory, etc.).
     - event.get('queryStringParameters'): Pulls query string (e.g., ?name=John).
     - Default fallback is "World" if no name is passed.
     - Returns HTTP-style response with a statusCode and a body.

#### GitHub Actions YAML – Build & Push Docker Image to Docker Hub

- .github/workflows/docker-build-push.yml

```
name: Build and Push Docker Image
on:
  push:
    branches:
      - main
  workflow_dispatch:
env:
  IMAGE_NAME: myapp
  DOCKER_HUB_USERNAME: ${{ secrets.DOCKER_HUB_USERNAME }}
  DOCKER_HUB_PASSWORD: ${{ secrets.DOCKER_HUB_PASSWORD }}
jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout Code
      uses: actions/checkout@v3
    - name: Log in to Docker Hub
      run: echo "${DOCKER_HUB_PASSWORD}" | docker login -u "${DOCKER_HUB_USERNAME}" --password-stdin
    - name: Build Docker Image
      run: docker build -t ${DOCKER_HUB_USERNAME}/${IMAGE_NAME}:latest .
    - name: Push Docker Image
      run: docker push ${DOCKER_HUB_USERNAME}/${IMAGE_NAME}:latest
```

#### Deployment Manifest (deployment.yaml)
- Defines the application’s workload, including containers, replicas, and resources.

```apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: my-container-image:latest
          ports:
            - containerPort: 80
```

#### Service Manifest (service.yaml)
- Exposes the deployment inside or outside the cluster.

```apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer  # Can be ClusterIP, NodePort, or LoadBalancer
```

#### Ingress Manifest (ingress.yaml)
- If using an NGINX Ingress Controller, define ingress rules.

```apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-app-service
                port:
                  number: 80
```
