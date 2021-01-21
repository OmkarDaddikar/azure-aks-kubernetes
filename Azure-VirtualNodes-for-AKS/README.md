# Azure Kubernetes Service AKS Virtual Nodes (Serverless)

## Step-01: Introduction
- What is [Virtual Kubelet](https://github.com/virtual-kubelet/virtual-kubelet)?
    - Virtual Kubelet is an open source **Kubernetes kubelet implementation** that acts as a kubelet for the purposes of connecting Kubernetes to other APIs. 
    - This allows the **k8s worker nodes** to be backed by other services like **Azure ACI and AWS Fargate**
    - The primary scenario for VK is enabling the extension of the Kubernetes API into **serverless container** platforms like Azure ACI and AWS Fargate

- What is [Azure Container Instances - ACI](https://docs.microsoft.com/en-us/azure/container-instances/)?
    - Azure Container Instances (ACI) provide a **hosted environment** for running containers in Azure. 
    - When using ACI, there is **no need to manage** the **underlying compute infrastructure**, Azure handles this management for you. 
    - When running containers in ACI, **you are charged by the second for each running container**

- What are [AKS Virtual Nodes](https://docs.microsoft.com/en-us/azure/aks/virtual-nodes-portal)?
We can run our Kubernetes workloads on **Serverless Infrastructure** of Azure which is called **Virtual Nodes**
### Advantages
- Scale our applications rapidly **without any limitations**
- **Quick Provisioning** of pods  using Virtual Nodes when compared to cluster autoscaler. 
- **Cluster Autoscaler** need to provision the Nodes in a managed node pool first then only **Kubernetes can schedule pods** on those newly  provisioned nodes.
### Limitations (Huge and Many)
- https://docs.microsoft.com/en-us/azure/aks/virtual-nodes-cli#known-limitations
**Important Note:** Virtual nodes require AKS clusters with [Azure CNI networking](https://docs.microsoft.com/en-us/azure/aks/configure-azure-cni)

[![Image](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-kubernetes-service-virtual-nodes.jpg "Azure AKS Kubernetes")](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-kubernetes-service-virtual-nodes.jpg)

## Step-02: Create a new cluster using Azure Management Console
- **Basics**
  - **Subscription:** Free Trial or Pay-as-you-go
  - **Resource Group:** Create New Resource Group: aks-preprod
  - **Kubernetes Cluster Name:** akspreprod
  - **Region:** (US) Central US
  - **Kubernetes Version:** Select what ever is latest stable version
  - **Node Size:** Standard DS2 v2 (Default one)
  - **Node Count:** 1
- **Node Pools**
  - **Virtual Nodes:** Enabled
  - leave to defaults
- **Authentication**
  - Authentication method: 	System-assigned managed identity
  - Rest all leave to defaults
- **Networking**
  - **Network Configuration:** Advanced
  - **Network Policy:** Azure
  - Rest all leave to defaults
- **Integrations**
  - **Azure Container Registry:** None
  - leave to defaults
- **Tags**
  - leave to defaults
- **Review + Create**
  - Click on **Create**


## Step-03: Verify Nodes & ACI
```
# Configure Command Line Credentials
az aks get-credentials --name akspreprod --resource-group aks-preprod

# Verify Nodes
kubectl get nodes 
kubectl get nodes -o wide

# Verify aci-connector-linux
kubectl get pods -n kube-system

# Verify logs of ACI Connector Linux
kubectl logs -f $(kubectl get po -n kube-system | egrep -o 'aci-connector-linux-[A-Za-z0-9-]+') -n kube-system
```
- We should see `virtual-node-aci-linux` node also listed for `kubectl get nodes` output
- **Sample Output**
```
kubectl get nodes
NAME                                STATUS   ROLES   AGE   VERSION
aks-agentpool-87689508-vmss000000   Ready    agent   24m   v1.17.11
virtual-node-aci-linux              Ready    agent   21m   v1.14.3-vk-azure-aci-v1.2.1.1
```

## Step-04: Update Deployment Manifest to Schedule Pod on Virtual Nodes
- The below section should be added in Deployment for Azure AKS to schedule the pod on Azure Virtual Nodes
- Review the manifests
```yaml
      nodeSelector:
        kubernetes.io/role: agent
        beta.kubernetes.io/os: linux
        type: virtual-kubelet
      tolerations:
      - key: virtual-kubelet.io/provider
        operator: Exists
      - key: azure.com/aci
        effect: NoSchedule
```

## Step-05: Deploy Application Manifests
```
# Deploy
kubectl apply -f kube-manifests/

# Verify pods
kubectl get pods -o wide

# Get Public IP
kubectl get svc

# Access Application
http://<Public-ip-captured-from-get-service>
```

## Step-06: Scale the Deployment 
```
# List Deployments
kubectl get deploy

# Scale the Deployment to 10 Replicas
kubectl scale --replicas=10 deployment awebsite-deployment

# List Pods
kubectl get pods
```

## References
- [Azure Virtual Nodes - Limitations](https://docs.microsoft.com/en-us/azure/aks/virtual-nodes-cli#known-limitations)
- [Virtual kubelet - Referece - 1](https://github.com/virtual-kubelet/virtual-kubelet)
- [Virtual kubelet - Referece - 2](https://github.com/virtual-kubelet/azure-aci/blob/master/README.md)
- [Virtual Node Autoscale - Optional & legacy](https://github.com/Azure-Samples/virtual-node-autoscale)