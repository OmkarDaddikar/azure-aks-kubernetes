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
| Persistent Volume Claim | 01-Persistent-Volume-Claim.yml   |
| Config Map  | 02-UserManagement-ConfigMap.yml  |
| Deployment, Environment Variables, Volumes, VolumeMounts  | 03-Mysql-Deployment.yml  |
| ClusterIP Service  | 04-Mysql-ClusterIP-Service.yml  |

## Step-02: Create following Kubernetes Manifests
Use AKS Provisioned Azure Disks
- Copy all templates from previous section
- Remove Storage Class Manifest
- **Question-1:** Why do we need to remove storage class Manifests?
- Azure AKS provisions two types of storage classes well in advance during the cluster creation process
  - managed-premium
  - default-
- We can leverage Azure AKS provisioned disk storage classes instead of what we created manually.
- **Question-2:** If that is the case why did we use custom storate class in previous section?
- That is for us to learn the `kind: StorageClass` concept.  

## Step-03: Review PVC Manifest 01-persistent-volume-claim.yml
- Primarily we are going to focus on `storageClassName: managed-premium` which tells us that we are going to use Azure AKS provisioned Azure Disks storage class.
```yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: azure-managed-disk-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: managed-premium 
  resources:
    requests:
      storage: 5Gi  
```

## Step-04: Deploy and Test
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
```

## Step-05: Connect to MySQL Database
```
# Connect to MYSQL Database
kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -pdbpassword11

# Verify usermgmt schema got created which we provided in ConfigMap
mysql> show schemas;
```

## Step-06: References & Storage Best Practices
- https://docs.microsoft.com/en-us/azure/aks/concepts-storage
- https://docs.microsoft.com/en-us/azure/aks/operator-best-practices-storage
- https://docs.microsoft.com/en-us/azure/aks/azure-disks-dynamic-pv
- https://kubernetes.io/docs/concepts/storage/persistent-volumes/
- https://kubernetes.io/docs/concepts/storage/storage-classes/#azure-disk
- https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#storageclass-v1-storage-k8s-io