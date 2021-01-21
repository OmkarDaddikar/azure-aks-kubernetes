# Azure AKS Storage - Azure Disks

[![Image](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-aks-storage.jpg "Azure AKS Kubernetes")](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-aks-storage.jpg)

## How we are going to use Azure Storage for Applications deployed on AKS for persistent Storage?

[![Image](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-aks-storage-for-application.jpg "Azure AKS Kubernetes")](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-aks-storage-for-application.jpg)

- Volumes that are defined and created as part of the **pod lifecycle** only exist until the pod is deleted.
- Pods often expect their storage to remain if a pod is **rescheduled** on a different host during a maintenance event, especially in StatefulSets.
- A **persistent volume (PV)** is a storage resource created and managed by the Kubernetes API that can exist beyond the lifetime of an individual pod.
- For AKS, **Azure Disks or Azure Files** are used to provide the PersistentVolume.
