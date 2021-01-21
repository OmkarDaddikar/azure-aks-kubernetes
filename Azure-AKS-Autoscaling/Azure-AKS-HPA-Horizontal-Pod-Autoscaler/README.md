# Azure AKS - Horizontal Pod Autoscaling (HPA)
## Step-01: Introduction
- In a very simple note Horizontal Scaling means **increasing and decreasing** the number of **Replicas (Pods)**.
- HPA **automatically scales** the number of pods in a deployment, replication controller, or replica set, stateful set based on that resource's **CPU utilization**.
- This can help our applications **scale out to meet increased demand** or **scale in when resources are not needed**, thus freeing up your worker nodes for other applications. 
- When we set a **target CPU utilization percentage**, the HPA scales our application in or out to try to meet that **target**.
- HPA needs **Kubernetes metrics server** to verify CPU metrics of a pod. 
- We **do not need** to **deploy or install the HPA** on our cluster to begin scaling our applications, its out of the box available as a **default** Kubernetes API resource. 

## How HPA Works?
[![Image](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-kubernetes-service-autoscaling-hpa-1.jpg "Azure AKS Kubernetes")](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-kubernetes-service-autoscaling-hpa-1.jpg)

## How HPA Configured?
[![Image](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-kubernetes-service-autoscaling-hpa-2.jpg "Azure AKS Kubernetes")](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-kubernetes-service-autoscaling-hpa-2.jpg)

## Step-02: Review Deploy our Application
```
# Deploy
kubectl apply -f kube-manifests

# List Pods, Deploy & Service
kubectl get pod
kubect get svc

# Access Application (Only if our Cluster is Public Subnet)
http://<PublicIP-from-Get-SVC-Output>
```

## Step-03: Create a Horizontal Pod Autoscaler resource for the "hpa-demo-deployment" 
- This command creates an autoscaler that targets 20 percent CPU utilization for the deployment, with a minimum of one pod and a maximum of ten pods. 
- When the average CPU load is below 20 percent, the autoscaler tries to reduce the number of pods in the deployment, to a minimum of one. 
- When the load is greater than 20 percent, the autoscaler tries to increase the number of pods in the deployment, up to a maximum of ten
```
# HPA Imperative - Template
kubectl autoscale deployment <deployment-name> --cpu-percent=20 --min=1 --max=10

# HPA Imperative - Replace
kubectl autoscale deployment hpa-demo-deployment --cpu-percent=20 --min=1 --max=10

# HPA Declarative (Optional - If you use above imperative command this is just for reference)
kubectl apply -f HPA-Manifest.yml

# Describe HPA
kubectl describe hpa/hpa-demo-deployment 

# List HPA
kubectl get horizontalpodautoscaler.autoscaling/hpa-demo-deployment 
```

## Step-04: Create the load & Verify how HPA is working
```
# Generate Load (new Terminal)
kubectl run apache-bench -i --tty --rm --image=httpd -- ab -n 500000 -c 1000 http://hpa-demo-service-nginx.default.svc.cluster.local/ 

# List all HPA
kubectl get hpa

# List specific HPA
kubectl get hpa hpa-demo-deployment 

# Describe HPA
kubectl describe hpa/hpa-demo-deployment 

# List Pods
kubectl get pods
```

## Step-05: Cooldown / Scaledown
- Default cooldown period is 5 minutes. 
- Once CPU utilization of pods is less than 20%, it will starting terminating pods and will reach to minimum 1 pod as configured.

## Referencess
- [Azure AKS - Horizontal Pod Autoscaler](https://docs.microsoft.com/en-us/azure/aks/tutorial-kubernetes-scale#autoscale-pods)