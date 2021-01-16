# Azure DevOps Release Pipelines for Website Deployment
[![Image](https://github.com/OmkarDaddikar/azure-aks-kubernetes/Images/azure-devops-release-pipelines-for-azure-aks.jpg "Azure AKS Kubernetes")](https://github.com/OmkarDaddikar/azure-aks-kubernetes/Images/azure-devops-release-pipelines-for-azure-aks.jpg)

# Pre-requisits 
## Create AKS Production Cluster
- Create Kubernetes Cluster
- **Basics**
  - **Subscription:** Free Trial
  - **Resource Group:** Create New Resource-Group-Name: aks-prod
  - **Kubernetes Cluster Name:** prod
  - **Region:** (US) Central US
  - **Kubernetes Version:** 1.16v Select what ever is latest stable version
  - **Node Size:** Standard DS2 v2 (Default one) You can select any as per your requirement
  - **Node Count:** 1 You can increase count as per your requirement
- **Node Pools**
  - Rest all leave to defaults
- **Authentication**
  - **Authentication Method:** 	System-assigned managed identity
  - Rest all leave to defaults
- **Networking**
  - **Network Configuration:** Advanced
  - **Network Policy:** Azure
  - Rest all leave to defaults
- **Integrations**
  - **Azure Container Registry:** None
  - Rest all leave to defaults
- **Tags**
  - Rest all leave to defaults
- **Review + Create**
  - Click on **Create**

## Create AKS Pre-Production Cluster
- Create Kubernetes Cluster
- **Basics**
  - **Subscription:** Free Trial
  - **Resource Group:** Create New Resource-Group-Name: aks-preprod
  - **Kubernetes Cluster Name:** preprod
  - **Region:** (US) Central US
  - **Kubernetes Version:** 1.16v Select what ever is latest stable version
  - **Node Size:** Standard DS2 v2 (Default one) You can select any as per your requirement
  - **Node Count:** 1 You can increase count as per your requirement
- **Node Pools**
  - Rest all leave to defaults
- **Authentication**
  - **Authentication Method:** 	System-assigned managed identity
  - Rest all leave to defaults
- **Networking**
  - **Network Configuration:** Advanced
  - **Network Policy:** Azure
  - Rest all leave to defaults
- **Integrations**
  - **Azure Container Registry:** None
  - Rest all leave to defaults
- **Tags**
  - Rest all leave to defaults
- **Review + Create**
  - Click on **Create**

## Cloud Shell - Configure kubectl to connect to AKS Cluster
- Go to https://shell.azure.com
```
# Template
az aks get-credentials --resource-group <Resource-Group-Name> --name <Cluster-Name>
```
# Azure AKS Cluster Access
[![Image](https://github.com/OmkarDaddikar/azure-aks-kubernetes/Images/azure-kubernetes-service-access-multiple-clusters.jpg "Azure AKS Kubernetes")](https://github.com/OmkarDaddikar/azure-aks-kubernetes/Images/azure-kubernetes-service-access-multiple-clusters.jpg)
## Create Azure Container Registry
- Go to **Services** -> Container Registries
- Click on **Add**
- **Subscription:** <Subsciption Name>
- **Resource Group:** Create New Resource-Group-Name: aks-registry
- **Registry Name:** aksdevopsacr   (NAME should be unique across Azure Cloud)
- **Location:** Central US
- **SKU:** Premium
- Click on **Review + Create**
- Click on **Create**
[![Image](https://github.com/OmkarDaddikar/azure-aks-kubernetes/Images/azure-kubernetes-service-and-acr-nodepools.jpg "Azure AKS Kubernetes")](https://github.com/OmkarDaddikar/azure-aks-kubernetes/Images/azure-kubernetes-service-and-acr-nodepools.jpg)
## Step-01: Creating Namespaces in Pre-Production and Production AKS Cluster
```
# Command for Accessing Pre-Production AKS Cluster
az aks get-credentials --resource-group aks-preprod --name preprod

# View the Current Context for kubectl
kubectl config current-context

# Switch Context
kubectl config use-context preprod
```
```
# Create Namespaces dev and preprod
kubectl create ns dev
kubectl create ns preprod
```
```
# Verify Namespaces Created
kubectl get ns
```
```
# Command for Accessing Production AKS Cluster
az aks get-credentials --resource-group aks-prod --name prod

# View the Current Context for kubectl
kubectl config current-context

# Switch Context
kubectl config use-context prod
```
```
# Create Namespaces prod
kubectl create ns prod
```
```
# Verify Namespaces Created
kubectl get ns
```

## Step-02: Create Github Project and Check-In Code
### Create Github Repository in Github
- **Name:** azure-devops-github-acr-aks-website-deployment
- **Description:** Azure DevOps Website Deployment with AKS, Github and Azure Containter Registry
- **Repository Type:** Public / Private (Your choice)
- Click on **Create Repository**

### Clone Github Repository and Check-In Code
```
# Clone the Git Repository 
git clone https://github.com/OmkarDaddikar/azure-aks-kubernetes.git
cd azure-aks-kubernetes
```
Copy all files to our New Repository Folder `azure-aks-kubernetes`
```
# Do Local Commit
git add .
git commit -am "V1 Base Commit"

# Push to Remote Repository
git push --set-upstream origin master
```
## Step-02: Create DevOps Organization
- Go to
  - https://dev.azure.com/
  - Sign in to Azure DevOps
- Our Organization will be automatically created and if you want to manually create organization you can create one. 
- **Organization Name:** Website-Deployment

## Step-03 : Create DevOps Project
- **Project Name:** azure-devops-github-acr-aks-website-deployment
- **Project Description:** AKS CICD Pipelines with Github and Azure Container Registry ACR
- **Visibility:** Public / Private (Your choice)
- **Advanced:** Leave to defaults
  - **Version Control:** Git
  - **Work Item Process:** Basic

## Step-04: Create Service Connections for Development, Pre-Production and Production Namespaces in Kubernetes Cluster and  Azure Container Registry
### Development Service Connection
**Go to Project** -> **azure-devops-github-acr-aks-website-deployment** -> **Project Settings** -> **Pipelines** -> **Service Connections**
- Click on **New Service Connection**
- Choose a **Service or Connection Type:** Kubernetes
- **Authentication Method:** Azure Subscription
- **Username:** Azure Cloud Administrator
- **Password:** Azure Cloud Admin Password
- **Cluster:** preprod
- **Namespace:** dev
- **Service Connection Name:** dev-ns-k8s-aks-svc-conn
- **Description:** Dev Namespace AKS Cluster Service Connection
- **Security:** Grant access permission to all pipelines (default Checked)
- Click on **SAVE** 

### Pre-Production Service Connection
**Go to Project** -> **azure-devops-github-acr-aks-website-deployment** -> **Project Settings** -> **Pipelines** -> **Service Connections**
- Click on **New Service Connection**
- Choose a **Service or Connection Type:** Kubernetes
- **Authentication Method:** Azure Subscription
- **Username:** Azure Cloud Administrator
- **Password:** Azure Cloud Admin Password
- **Cluster:** preprod
- **Namespace:** preprod
- **Service Connection Name:** preprod-ns-k8s-aks-svc-conn
- **Description:** Pre-Production Namespace AKS Cluster Service Connection
- **Security:** Grant access permission to all pipelines (default Checked)
- Click on **SAVE** 

### Production Service Connection
**Go to Project** -> **azure-devops-github-acr-aks-website-deployment** -> **Project Settings** -> **Pipelines** -> **Service Connections**
- Click on **New Service Connection**
- Choose a **Service or Connection Type:** Kubernetes
- **Authentication Method:** Azure Subscription
- **Username:** Azure Cloud Administrator
- **Password:** Azure Cloud Admin Password
- **Cluster:** prod
- **Namespace:** prod
- **Service Connection Name:** prod-ns-k8s-aks-svc-conn
- **Description:** Production Namespace AKS Cluster Service Connection
- **Security:** Grant access permission to all pipelines (default Checked)
- Click on **SAVE** 

### Azure Container Registry Service Connection
 **Go to Project** -> **azure-devops-github-acr-aks-website-deployment** -> **Project Settings** -> **Pipelines** -> **Service Connections**
- Click on **New Service Connection**
- Choose a **Service or Connection Type:** Docker Registry
- **Registry Type:** Azure Container Registry
- **Authentication Method:** Azure Subscription
- **Username:** Azure Cloud Administrator
- **Password:** Azure Cloud Admin Password
- **Azure Container Registry:** aksdevopsacr
- **Service Connection Name:** aksdevopsacr-svc
- **Service Connection Description:** SVC for ACR named aksdevopsacr
- Click on **Save**
[![Image](https://github.com/OmkarDaddikar/azure-aks-kubernetes/Images/azure-devops-pipelines-key-concept.jpg "Azure AKS Kubernetes")](https://github.com/OmkarDaddikar/azure-aks-kubernetes/Images/azure-devops-pipelines-key-concept.jpg)
## Step-05: Task-1: Create a Build Pipeline and Publish Artifacts to Azure Pipelines
- Go to **Pipelines** -> Create **New Folder** -> Website-Deployment-Pipelines
- Go to **Pipelines** -> Create **New Pipeline**
- Where is your Code?: **Github**  
- **Select Repository:** azure-devops-github-acr-aks-website-deployment
  - Provide Github **Username** and **Password**
  - Click on **Approve and Install** for Repositories selected
- **Configure Your Pipeline:** Docker (Build and Push Image to Azure Container Registry)
- Select an **Azure Subscription:** <Subscription Name>
- **Container Registry:** aksdevopsacr
- **Image Name:** <Website-Name>
- **Dockerfile:** $(Build.SourcesDirectory)/Dockerfile
- Click on **Validate and Configure**
- Change **Pipeline Name:** BuildPushToACR-Publish-k8s-manifests-to-AzurePipelines.yml and Move to **Folder:** Website-Deployment-Pipelines

## Step-05: Task-2: Copy kube-manifests to Artifact Staging Directory
- Copy files from source directory to target directory
```yaml
## Publish Artifacts pipeline code in addition to Build and Push          
    - bash: echo Contents in System Default Working Directory; ls -R $(System.DefaultWorkingDirectory)        
    - bash: echo Before copying Contents in Build Artifact Directory; ls -R $(Build.ArtifactStagingDirectory)        
    # Task-2: Copy files (Copy files from a source folder to target folder)
    # Source Directory: $(System.DefaultWorkingDirectory)/kube-manifests
    # Target Directory: $(Build.ArtifactStagingDirectory)
    - task: CopyFiles@2
      inputs:
        SourceFolder: '$(System.DefaultWorkingDirectory)/kube-manifests'
        Contents: '**'
        TargetFolder: '$(Build.ArtifactStagingDirectory)'
        OverWrite: true
    # List files from Build Artifact Staging Directory - After Copy
    - bash: echo After copying to Build Artifact Directory; ls -R $(Build.ArtifactStagingDirectory) 
```

## Step-05: Task-3: Publish Artifacts to Azure Pipelines
```yaml
    # Task-3: Publish build artifacts (Publish build to Azure Pipelines)           
    - task: PublishBuildArtifacts@1
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: 'kube-manifests'
        publishLocation: 'Container'         
```
## Step-06: Save and Run the Build, Verify Build, Copy and Publish Task logs
- **Commit Message:** Build Push To ACR & Publish Kubernetes Manifests To Azure Pipelines
- Click on **Save and Run**
- Commit directly to master branch: check
- Build Task should pass. Verify logs
- Copy Task should pass.  Verify logs
- Publish Task should pass. Verify logs

## Step-07: Create Release Pipeline - Add Artifacts
- **Release Pipeline Name:** website-deployment-release-pipeline
### Add Artifact
- **Source Type:** Build
- **Project:** leave to default (azure-devops-github-acr-aks-website-deployment)
- **Source (Build Pipeline):** Website-Deployment-Pipelines\BuildPushToACR-Publish-k8s-manifests-to-AzurePipelines
- **Default Version:** Latest (auto-populated)
- **Source Alias:** leave to default (auto-populated)
- Click on **Add**

### Continuous Deployment Trigger
- **Continuous Deployment Trigger:** Enabled

## Step-08: Release Pipeline - Create Dev Stage
- Go to **Pipelines** -> **Releases**
- Create new **Release Pipeline**

### Create Development Pre-Production and Production
- **Stage Name:** Dev
- **Create Task** 
- **Agent Job:** Change to Ubunut Linux (latest)

### Add Task: Create Secret
- **Display Name:** Create Secret to allow Image Pull from ACR
- **Action:** create secret
- **Kubernetes Service Connection:** dev-ns-k8s-aks-svc-conn
- **Namespace:** dev
- **Type Of Secret:** dockerRegistry
- **Secret Name:** dev-aksdevopsacr-secret
- **Docker Registry Service Connection:** aksdevopsacr-svc
- Click on **SAVE** to save release
- **Commit Message:** Development Create Secret Task Updated

### Add Task: Deploy to Kubernetes
- **Display Name:** Deploy to AKS
- **Action:** deploy
- **Kubernetes Service Connection:** dev-ns-k8s-aks-svc-conn
- **Namespace:** dev
- **Strategy:** None
- **Manifest:** Select Deployment-and-LoadBalancer-Service.yml from build artifacts
```
# Sample Value for Manifest after adding it
Manifest: $(System.DefaultWorkingDirectory)/BuildPushToACR-Publish-k8s-manifests-to-AzurePipelines/kube-manifests/Deployment-and-LoadBalancer-Service.yml
```
- **Container:** aksdevopsacr.azurecr.io/<Website Name>:$(Build.SourceVersion)
- **ImagePullSecrets:** dev-aksdevopsacr-secret
- Click on **SAVE** to save release
- **Commit Message:** Development Deploy to AKS Task Updated

## Step-9: Create Pre-Production and Production Release Stages
- Create Pre-Production and Production Stages
- Add Email Approvals
- Click on **SAVE** to save release

### Clone Development Stage to Create Pre-Production Stage
- Go to **Releases** -> **website-deployment-release-pipeline** -> Edit
- Select **Dev Stage** -> Add -> **Clone Stage**
- **Stage Name:** Pre-Production

#### Task-1: Create Secret
- **Kubernetes Service Connection:** preprod-ns-k8s-aks-svc-conn
- **Namespace:** preprod
- **Secret Name:** preprod-aksdevopsacr-secret
- Click **SAVE**
- **Commit Message:** Pre-Production Create Secret Task Updated

#### Task-2: Deploy to AKS
- **Kubernetes Service Connection:** preprod-ns-k8s-aks-svc-conn
- **Namespace:** preprod
- **ImagePullSecrets:** preprod-aksdevopsacr-secret
- Click **SAVE**
- **Commit Message:** Pre-Production Deploy to AKS Task Updated
```
# Sample Value for Manifest after adding it
Manifest: $(System.DefaultWorkingDirectory)/BuildPushToACR-Publish-k8s-manifests-to-AzurePipelines/kube-manifests/Deployment-and-LoadBalancer-Service.yml
```
- **Container:** aksdevopsacr.azurecr.io/<Website Name>:$(Build.SourceVersion)
- **ImagePullSecrets:** preprod-aksdevopsacr-secret
- Click on **SAVE** to save release
- **Commit Message:** Pre-Production Deploy to AKS Task Updated

### Clone Pre-Production Stage to Create Production Stage
- Go to **Releases** -> **website-deployment-release-pipeline** -> **Edit**
- Select **Pre-Production Stage** -> Add -> **Clone Stage**
- **Stage Name:** Production

#### Task-1: Create Secret
- **Kubernetes Service Connection:** prod-ns-k8s-aks-svc-conn
- **Namespace:** prod
- **Secret Name:** prod-aksdevopsacr-secret
- Click **SAVE**
- **Commit Message:** Production Create Secret Task Updated

#### Task-2: Deploy to AKS
- **Kubernetes Service Connection:** prod-ns-k8s-aks-svc-conn
- **Namespace:** prod
- **ImagePullSecrets:** prod-aksdevopsacr-secret
- Click **SAVE**
- **Commit Message:** Production Deploy to AKS Task Updated
```
# Sample Value for Manifest after adding it
Manifest: $(System.DefaultWorkingDirectory)/BuildPushToACR-Publish-k8s-manifests-to-AzurePipelines/kube-manifests/Deployment-and-LoadBalancer-Service.yml
```
- **Container:** aksdevopsacr.azurecr.io/<Website Name>:$(Build.SourceVersion)
- **ImagePullSecrets:** prod-aksdevopsacr-secret
- Click on **SAVE** to save release
- **Commit Message:** Production Deploy to AKS Task Updated

## Step-10: Make Changes to the Code and Test Continuous Integration and Continuous Deployment Pipeline
- View Build Logs
- View Development Release logs
- Access Application after successful deployment
- Approve deployment at pre-production stages
- View Pre-Production Release logs
- Access Application after successful deployment
- Approve deployment at production stages
- View Production Release logs
- Access Application after successful deployment

# Get Public IP for Development Namespace
```
# Command for Accessing Pre-Production AKS Cluster
az aks get-credentials --resource-group aks-preprod --name preprod

# View the Current Context for kubectl
kubectl config current-context

# Switch Context
kubectl config use-context preprod

#  Get Service List in dev Namespace
kubectl get svc -n dev
```
# Get Public IP for Pre-Production(Staging) Namespace
```
# Command for Accessing Pre-Production AKS Cluster
az aks get-credentials --resource-group aks-preprod --name preprod

# View the Current Context for kubectl
kubectl config current-context

# Switch Context
kubectl config use-context preprod

#  Get Service List in preprod Namespace
kubectl get svc -n preprod
```
# Get Public IP for Production Namespace
```
# Command for Accessing Production AKS Cluster
az aks get-credentials --resource-group aks-prod --name prod

# View the Current Context for kubectl
kubectl config current-context

# Switch Context
kubectl config use-context prod

#  Get Service List in prod Namespace
kubectl get svc -n prod
```
# Access Application
```
http://<Public-IP-from-Get-Service-Output>

# Website is Live with Below Domain Name
http://pilankarmasale.com/

```
