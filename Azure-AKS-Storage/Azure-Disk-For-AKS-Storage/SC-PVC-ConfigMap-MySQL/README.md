# AKS Storage -  Storage Classes, Persistent Volume Claims
## Step-01: Introduction
- Azure Disk Storage offers high-performance, highly durable block storage for our mission- and business-critical workloads
- We can mount these volumes as devices on our Virtual Machines & Container instances. 
### Cost-effective Storage
- Built-in bursting capabilities to handle unexpected traffic and process batch jobs cost-effectively
### Unmatched Resiliency
- 0 percent annual failure rate for consistent enterprise-grade durability
### Seamless Scalability
- Dynamic scaling of disk performance on Ultra Disk Storage without disruption
### Built-in Security
- Automatic encryption to help protect your data using Microsoft-managed keys or your own

[![Image](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-aks-storage-for-UserManagement.jpg "Azure AKS Kubernetes")](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-aks-storage-for-UserManagement.jpg)

We are going to create a MySQL Database with persistence storage using **Azure Disks** 

| Kubernetes Object  | YAML File |
| ------------- | ------------- |
| Storage Class  | 01-Storage-Class.yml |
| Persistent Volume Claim | 02-Persistent-Volume-Claim.yml   |
| Config Map  | 03-UserManagement-ConfigMap.yml  |
| Deployment, Environment Variables, Volumes, VolumeMounts  | 04-Mysql-Deployment.yml  |
| ClusterIP Service  | 05-Mysql-ClusterIP-Service.yml  |

## Step-02: Create following Kubernetes Manifests
### Create Storage Class Manifest
- https://kubernetes.io/docs/concepts/storage/storage-classes/#volume-binding-mode
- https://kubernetes.io/docs/concepts/storage/storage-classes/#azure-disk
```
# Create Storage Class & PVC
kubectl apply -f kube-manifests/01-Storage-Class.yml
kubectl apply -f kube-manifests/02-Persistent-Volume-Claim.yml

# List Storage Classes
kubectl get sc

# List PVC
kubectl get pvc 

# List PV
kubectl get pv
```
### Create ConfigMap Manifest
- We are going to create a `usermgmt` database schema during the mysql pod creation time which we will leverage when we deploy User Management Microservice. 

### Create MySQL Deployment Manifest
- Environment Variables
- Volumes
- Volume Mounts

### Create MySQL ClusterIP Service Manifest
- At any point of time we are going to have only one mysql pod in this design so `ClusterIP: None` will use the `Pod IP Address` instead of creating or allocating a separate IP for `MySQL Cluster IP service`.   

## Step-03: Create MySQL Database with all above Manifests
```
# Create MySQL Database
kubectl apply -f kube-manifests/

# List Storage Classes
kubectl get sc

# List PVC
kubectl get pvc 

# List PV
kubectl get pv

# List pods
kubectl get pods 

# List pods based on  label name
kubectl get pods -l app=mysql
```

## Step-04: Connect to MySQL Database
```
# Connect to MYSQL Database
kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -pdbpassword11

# Verify usermgmt schema got created which we provided in ConfigMap
mysql> show schemas;
```

## Step-05: References & Storage Best Practices
- https://docs.microsoft.com/en-us/azure/aks/concepts-storage
- https://docs.microsoft.com/en-us/azure/aks/operator-best-practices-storage
- https://docs.microsoft.com/en-us/azure/aks/azure-disks-dynamic-pv
- https://kubernetes.io/docs/concepts/storage/persistent-volumes/
- https://kubernetes.io/docs/concepts/storage/storage-classes/#azure-disk
- https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#storageclass-v1-storage-k8s-io
