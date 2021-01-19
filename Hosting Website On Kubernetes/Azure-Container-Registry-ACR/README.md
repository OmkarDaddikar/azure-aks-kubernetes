# Integrate Azure Container Registry ACR with AKS

## Step-01: Introduction
- Azure Container Registry is a **managed, private Docker registry** service 
- We can **create and maintain** Azure container registries to store and manage our  private Docker container images.
- With ACR, we can simplify our **container lifecycle management**.
- **Geo-replication** to efficiently manage a single registry across multiple regions
- **Automated container building** and patching including base image updates and task scheduling
- **Integrated security** with Azure Active Directory (Azure AD) authentication, role-based access control, **Docker Content Trust** and virtual network integration

[![Image](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-kubernetes-service-and-acr.jpg "Azure AKS Kubernetes")](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-kubernetes-service-and-acr.jpg)

[![Image](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-container-registry-pricing-tiers.jpg "Azure AKS Kubernetes")](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-container-registry-pricing-tiers.jpg)

## Step-02: Create Azure Container Registry
 Go to **Services** -> Container Registries
- Click on **Add**
- **Subscription:** <Subsciption Name>
- **Resource Group:** Create New Resource-Group-Name: aks-registry
- **Registry Name:** aksdevopsacr   (NAME should be unique across Azure Cloud)
- **Location:** Central US
- **SKU:** Premium
- Click on **Review + Create**
- Click on **Create**

## Step-03: Build Docker Image Locally
- Review Docker Manigests 
```

# Docker Build
docker build -t pilankarmasale .

# List Docker Images
docker images ls | grep -i pilankarmasale
```

## Step-04: Run Docker Container Locally and Test
```
# Run locally and Test
docker run --name pilankarmasale -d -p 80:80 pilankarmasale:latest

# Access Application locally
http://localhost

# Stop Docker Image
docker stop pilankarmasale
```

## Step-05: Enable Docker Login for ACR Repository 
- Go to **Services** -> **Container Registries** -> aksdevopsacr
- Go to **Access Keys**
- Click on **Enable Admin User**
- Make a note of Username and password

## Step-06: Push Docker Image to ACR

### Build, Test Locally, Tag and Push to ACR
```
# Export Command
export ACR_REGISTRY=aksdevopsacr.azurecr.io
export ACR_NAMESPACE=app
export ACR_IMAGE_NAME=pilankarmasale
export ACR_IMAGE_TAG=v1
echo $ACR_REGISTRY, $ACR_NAMESPACE, $ACR_IMAGE_NAME, $ACR_IMAGE_TAG

# Login to ACR
docker login $ACR_REGISTRY

# Tag
docker tag pilankarmasale:v1  $ACR_REGISTRY/$ACR_NAMESPACE/$ACR_IMAGE_NAME:$ACR_IMAGE_TAG
It replaces as below
docker tag pilankarmasale:v1 aksdevopsacr.azurecr.io/app/pilankarmasale:v1

# List Docker Images to verify
docker images ls | grep -i aksdevopsacr.azurecr.io/app/pilankarmasale:v1
docker images $ACR_REGISTRY/$ACR_NAMESPACE/$ACR_IMAGE_NAME:$ACR_IMAGE_TAG

# Push Docker Images
docker push aksdevopsacr.azurecr.io/app/pilankarmasale:v1
docker push $ACR_REGISTRY/$ACR_NAMESPACE/$ACR_IMAGE_NAME:$ACR_IMAGE_TAG
```
### Verify Docker Image in ACR Repository
- Go to **Services** -> **Container Registries** -> **aksdevopsacr**
- Go to **Repositories** -> **app/pilankarmasale**

[![Image](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-kubernetes-service-and-acr-nodepools.jpg "Azure AKS Kubernetes")](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-kubernetes-service-and-acr-nodepools.jpg)

## Step-07: Create Service Principal to access Azure Container Registry
- Review file: shell-script/generate-service-principal.sh
- Update ACR_NAME with your container registry name
- Update SERVICE_PRINCIPAL_NAME as desired
```sh
#!/bin/bash

# Modify for your environment.
# ACR_NAME: The name of your Azure Container Registry
# SERVICE_PRINCIPAL_NAME: Must be unique within your AD tenant
#ACR_NAME=<container-registry-name>
ACR_NAME=aksdevopsacr
SERVICE_PRINCIPAL_NAME=acr-sp-demo

# Obtain the full registry ID for subsequent command args
ACR_REGISTRY_ID=$(az acr show --name $ACR_NAME --query id --output tsv)

# Create the service principal with rights scoped to the registry.
# Default permissions are for docker pull access. Modify the '--role'
# argument value as desired:
# acrpull:     pull only
# acrpush:     push and pull
# owner:       push, pull, and assign roles
SP_PASSWD=$(az ad sp create-for-rbac --name http://$SERVICE_PRINCIPAL_NAME --scopes $ACR_REGISTRY_ID --role acrpull --query password --output tsv)
SP_APP_ID=$(az ad sp show --id http://$SERVICE_PRINCIPAL_NAME --query appId --output tsv)

# Output the service principal's credentials; use these in your services and
# applications to authenticate to the container registry.
echo "Service principal ID: $SP_APP_ID"
echo "Service principal password: $SP_PASSWD"
```

## Step-08: Create Image Pull Secret
```
# Template
kubectl create secret docker-registry <secret-name> \
    --namespace <namespace> \
    --docker-server=<container-registry-name>.azurecr.io \
    --docker-username=<service-principal-ID> \
    --docker-password=<service-principal-password>

# Replace
kubectl create secret docker-registry aksdevopsacr-secret \
    --namespace default \
    --docker-server=aksdevopsacr.azurecr.io \
    --docker-username=80beacfe-7176-4ff5-ad22-dbb15528a9a8 \
    --docker-password=0zjUzGzSx3_.xi1SC40VcWkdVyl8Ml8QNj    

# List Secrets
kubectl get secrets    
```

## Step-09: Review, Update & Deploy to AKS & Test
### Update Deployment Manifest with Image Name, ImagePullSecrets
```yaml
    spec:
      containers:
        - name: pilankar-masale
          image: aksdevopsacr.azurecr.io/app/pilankarmasale:v1
          imagePullPolicy: Always
          ports:
            - containerPort: 80
      imagePullSecrets:
        - name: aksdevopsacr-secret           
```

## Step-10: Configure ACR integration for existing AKS clusters
```
#Set ACR NAME
export ACR_NAME=aksdevopsacr
echo $ACR_NAME

# Template
az aks update -n myAKSCluster -g myResourceGroup --attach-acr <acr-name>

# Replace Cluster, Resource Group and ACR Repo Name
az aks update -n aksprod -g aks-prod --attach-acr $ACR_NAME
```

### Deploy to AKS and Test
```
# Deploy
kubectl apply -f kube-manifests/

# List Pods
kubectl get pods

# Describe Pod
kubectl describe pod <pod-name>

# Get Load Balancer IP
kubectl get svc

# Access Application
http://<External-IP-from-get-service-output>
```

## References
- [Azure Container Registry Authentication - Options](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-authentication)
- [Pull images from an Azure container registry to a Kubernetes cluster](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-auth-kubernetes)








