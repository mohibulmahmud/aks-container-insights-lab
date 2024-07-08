# Lab | Container Insights with AKS: A Hands-On Lab Using Azure CLI

## Overview
This hands-on lab will help you set up a working Container Insights environment on an AKS cluster using Azure CLI and monitor logs in a Log Analytics Workspace.

## Process

### 1. Create Your Azure Resources
Please create the following resources in your subscription to get started:

- Resource Group
- Log Analytics Workspace
- AKS Cluster

#### Commands:
```sh
# Create a New Resource Group in West US
az group create --name akscontainerinsightsrg-westus --location westus

# Create a Log Analytics Workspace in the New Resource Group
az monitor log-analytics workspace create --resource-group akscontainerinsightsrg-westus --workspace-name akscontainerinsightsworkspace --location westus

# Create an AKS Cluster in the New Resource Group with Monitoring addon Enabled

# The first command retrieves the ID of a specified Log Analytics workspace and stores it in the workspaceId variable.
workspaceId=$(az monitor log-analytics workspace show --resource-group akscontainerinsightsrg-westus --workspace-name akscontainerinsightsworkspace --query id -o tsv)

# The second command creates an AKS cluster with monitoring enabled, linking it to the Log Analytics workspace using the retrieved ID. This setup integrates Azure Monitor for containers with the AKS cluster.
az aks create --resource-group akscontainerinsightsrg-westus --name akscontainerinsightscluster --node-count 1 --enable-addons monitoring --generate-ssh-keys --workspace-resource-id $workspaceId
```

### 2. Connect to Your AKS Cluster
In this step, we'll connect to the AKS Cluster via the Azure CLI, which is easiest to perform in the Cloud Shell.

#### Commands:
```sh
# Get AKS Credentials
az aks get-credentials --resource-group akscontainerinsightsrg-westus --name akscontainerinsightscluster

# Create a Namespace (Optional but recommended)
kubectl create namespace sample-app
```

### 3. Deploy a Sample Application
Deploy a sample Nginx application.

#### Commands:
```sh
# Deploy a Sample Nginx Application
kubectl create deployment sample-nginx --image=nginx --namespace=sample-app
```

### 4. Expose the Deployment
Expose the deployment to create a LoadBalancer service which makes it accessible from outside.

#### Commands:
```sh
# Expose the Deployment
kubectl expose deployment sample-nginx --port=80 --target-port=80 --type=LoadBalancer --name=sample-nginx-service --namespace=sample-app
```

### 5. Generate Logs
Generate some logs by accessing the sample application.

#### Commands:
```sh
# Verify the Service to Get the External IP Address
kubectl get svc --namespace=sample-app

# Access the Application via Web Browser
externalIp=$(kubectl get svc sample-nginx-service --namespace=sample-app -o jsonpath="{.status.loadBalancer.ingress[0].ip}")
echo "Access the application at: http://$externalIp"
curl http://$externalIp
```

### 6. Monitor Logs in Log Analytics Workspace
Monitor the logs in the Log Analytics Workspace.

#### Steps:
1. Navigate to the Azure portal.
2. Go to your Log Analytics workspace (akscontainerinsightsworkspace).
3. Select "Logs" under the General section.
4. Run a query to view the logs, such as:
   ```kql
   ContainerLogV2
   | limit 10
   ```

### 7. Optional: Clean Up Resources
If you no longer need the resources in the resource group, you can delete it to avoid incurring charges.

#### Command:
```sh
# Delete the Resource Group
az group delete --name akscontainerinsightsrg-westus --yes --no-wait
```

## Provide Feedback
We value your feedback! Please let us know if you have any suggestions or comments about this lab.
```
