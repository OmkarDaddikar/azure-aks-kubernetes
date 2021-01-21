# Azure AD Authentication for AKS Cluster
[![Image](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-kubernetes-service-ad-authentication-part-2.jpg "Azure AKS Kubernetes")](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-kubernetes-service-ad-authentication-part-2.jpg)

[![Image](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-kubernetes-service-ad-authentication-part-2.jpg "Azure AKS Kubernetes")](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-kubernetes-service-ad-authentication-part-2.jpg)

## Step-01: Create Azure AD Group and User in Azure AD 
### Create Azure AD Group 
- **Group Type:** Security 
- **Group Name:** K8sUsers
- **Group Description:** AKS Cluster Users who has full access to Kubernetes Clusters 
- Click on **Create**

### Create Azure AD User & Associate User to Group
- Create User in Azure Active Directory &  Associate User to **K8sUsers** group
- Go to **All Services** -> **Azure Active Directory** -> **Users** -> **New User**
- **Identity**
  - **Username:** user1
  - **Name:** user1 AKSUser
  - **First Name:** user1
  - **Last Name:** AKSUser
- **Password**
  - Let me create the password: check the radio button
  - **Initial Password:** @AKSUser123
- **Groups & Role**
  - **Groups:** K8sUsers
  - **Roles:** User
- Rest all leave to defaults
- Click on **Create**

### Complete First Time User Password Change
- Gather Full Username from AD
- **URL:** https://portal.azure.com
- **Username:** user1@omkardaddikargmail.onmicrosoft.com 
- **Current Password:** @AKSUser123
- **New Password:** @AKSADAuth123
- **Confirm Password:** @AKSADAuth123

### Final Username and Password
- **Username:** user1@omkardaddikargmail.onmicrosoft.com 
- **Password:** @AKSADAuth123

## Step-02: Enable AKS Cluster with AKS-Managed Azure Active Directory Feature
- Go to **All Services** -> **Kubernetes Services** -> **aksprod** -> **Settings** -> **Configuration**
- **AKS-Managed Azure Active Directory:** Select **Enabled** radio button
- **Admin Azure AD Groups:** K8sUsers
- Click on **SAVE**

## Step-03: Access an Azure AD enabled AKS Cluster using Azure AD User
- **Important Note:** Once we do **devicelogin** credentials are cached for all subsequent kubectl commands.
```
# Configure kubectl
az aks get-credentials --resource-group aks-prod --name aksprod --overwrite-existing
```

```
# View Cluster Information
kubectl cluster-info
```

**URL:** https://microsoft.com/devicelogin
**Code:** H8VP9YE7F (Sample)(View on terminal)
**Username:** user1@omkardaddikargmail.onmicrosoft.com
**Password:** @AKSADAuth123

```
# List Nodes
kubectl get nodes

# List Pods
kubectl get pods -n kube-system

# List Everything
kubectl get all --all-namespaces
```

## Step-04: How to re-login with different user for kubectl ?
- **Important Note:** The moment we change the `$HOME/.kube/config` you will be re-prompted for Azure Device Login for all kubectl commands
- We need to overwrite the $HOME/.kube/config to re-login with different user or same user
```
# Overwrite kubectl credentials 
az aks get-credentials --resource-group aks-prod --name aksprod --overwrite-existing

# View kubectl config (Observe aksdemo3 user)
kubectl config view 
```

```
# List Nodes
kubectl get nodes
```

**URL:** https://microsoft.com/devicelogin
**Code:** H8VP9YE7F (Sample)
**Username:** user2@omkardaddikargmail.onmicrosoft.com
**Password:** @AKSADAuth123

```
# View kubectl config (Observe aksdemo3 user - Access Token & Refresh token)
kubectl config view 
```

## Step-05: How to bypass or Override AD Authentication and use k8s admin?
- If we have issues with AD Users or Groups and want to override that we can use **--admin** to override and directly connect to AKS Cluster
```
# Template
az aks get-credentials --resource-group <ResourceGroupName> --name <ManagedClusterName> --admin

# Replace RG and Cluster Name
az aks get-credentials --resource-group aks-prod --name aksprod --admin

# List Nodes
kubectl get nodes

# List Pods
kubectl get pods -n kube-system
```

## References
- [AKS Managed AAD](https://docs.microsoft.com/en-us/azure/aks/managed-aad)
- [Azure AD RBAC](https://docs.microsoft.com/en-us/azure/aks/azure-ad-rbac)
- [Azure AKS Identity](https://docs.microsoft.com/en-us/azure/aks/concepts-identity)