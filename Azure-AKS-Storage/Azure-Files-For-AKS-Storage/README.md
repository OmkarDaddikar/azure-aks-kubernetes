# Azure Files for AKS Storage
## Step-01: Introduction
- These are simple, secure and **fully managed** cloud file shares
- We can secure **data at rest and in-transit** using SMB 3.0 and HTTPS
- We can create **high-performance file shares** using the Premium Files storage tier
- We can replace or supplement **on-premises file servers**
- **Scripting and tooling:** **PowerShell** cmd lets and **Azure CLI** can be used to create, mount, and manage - Azure file shares as part of the administration of Azure applications. 
- We can create and manage Azure file shares using **Azure Portal** and **Azure Storage Explorer**.
- For Application workloads we can use for use cases like **Static Content Storage**, shared **configuration access** to multiple JVMs etc.

[![Image](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-aks-filestorage-for-application.jpg "Azure AKS Kubernetes")](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-aks-filestorage-for-application.jpg)

- We are going to write a Deployment Manifest for NGINX Application which will have its static content served from Azure File Shares in app1 folder
- We are going to mount the file share to a specific path mountPath: "/usr/share/nginx/html/app1" in the Nginx container
### kube-manifests-sc-pvc: Custom Storage Class
- We will define our own custom storage class with desired permissions
    - Standard_LRS - standard locally redundant storage (LRS)
    - Standard_GRS - standard geo-redundant storage (GRS)
    - Standard_ZRS - standard zone redundant storage (ZRS)
    - Standard_RAGRS - standard read-access geo-redundant storage (RA-GRS)
    - Premium_LRS - premium locally redundant storage (LRS)

### kube-manifests-pvc: AKS defined default storage class
- With default AKS created storage classes only below two options are available for us.
    - Standard_LRS - standard locally redundant storage (LRS)
    - Premium_LRS - premium locally redundant storage (LRS)

**Important Note:** Azure Files support premium storage in AKS clusters that run Kubernetes 1.13 or higher, minimum premium file share is 100GB

## Step-02: Create or Review kube-manifests-v1 and Nginx Files
**Kube Manifests**
01-Storage-Class.yml
02-Persistent-Volume-Claim.yml
03-Nginx-Deployment.yml
04-Nginx-Service.yml

**nginx-files**
file1.html
file2.html
K8s Deployment Maniest - Core Item For Review

          volumeMounts:
            - name: my-azurefile-volume
              mountPath: "/usr/share/nginx/html/app1"
      volumes:
        - name: my-azurefile-volume
          persistentVolumeClaim:
            claimName: my-azurefile-pvc    
## Step-03: Deploy Kube Manifests sc-pvc
### Deploy
```
kubectl apply -f kube-manifests-sc-pvc/
```
### Verify SC, PVC, PV
```
kubectl get sc, pvc, pv
```
### Verify Pod
```
kubectl get pods
kubectl describe pod <pod-name>
```
### Get Load Balancer Public IP
```
kubectl get svc
```
### Access Application
```
http://<External-IP-from-get-service-output>
http://<External-IP-from-get-service-output>/app1/index.html
```
## Step-04: Upload Nginx Files to Azure File Share
Go to **Storage Accounts**
Select and Open storage account under **Resoure Group** <Resource-Group-Name>
In Overview, go to File Shares
Open File share with name which starts as kubernetes-dynamic-pv-xxxxxx
Click on Upload and Upload
file1.html
file2.html

## Step-05: Access Application & Test
```
http://<External-IP-from-get-service-output>/app1/file1.html
http://<External-IP-from-get-service-output>/app1/file2.html
```
## Step-06: Clean-Up
```
kubectl delete -f kube-manifests-sc-pvc/
```
## Step-07: Create or Review kube-manifests-pvc and Nginx Files
**Kube Manifests**
01-Persistent-Volume-Claim.yml
02-Nginx-Deployment.yml
03-Nginx-Service.yml

**nginx-files**
file1.html
file2.html

## Step-08: Deploy Kube Manifests V2
### Deploy
```
kubectl apply -f kube-manifests-pvc/
```
### Verify SC, PVC, PV
```
kubectl get sc, pvc, pv
```
### Verify Pod
```
kubectl get pods
kubectl describe pod <pod-name>
```
### Get Load Balancer Public IP
```
kubectl get svc
```
### Access Application
```
http://<External-IP-from-get-service-output>
```
## Step-09: Upload Nginx Files to Azure File Share
Go to **Storage Accounts**
Select and Open storage account under resoure group <Resource-Group-Name>
In Overview, go to File Shares
Open File share with name which starts as kubernetes-dynamic-pv-xxxxxx
Click on Upload and Upload
file1.html
file2.html

## Step-10: Access Application & Test
```
http://<External-IP-from-get-service-output>/app1/file1.html
http://<External-IP-from-get-service-output>/app1/file2.html
```
## Step-11: Clean-Up
```
kubectl delete -f kube-manifests-pvc/
```
## References
https://docs.microsoft.com/en-us/azure/aks/azure-files-dynamic-pv
