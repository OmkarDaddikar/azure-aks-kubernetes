# Kubernetes Namespaces - ResourceQuota - Declarative using YAML

## Introduction
- When several users or teams **share a cluster with a fixed number of nodes**, there is a concern that one team could use more than its fair share of resources.
- Resource quotas are a tool for administrators to **address this concern**.
- A resource quota, defined by a ResourceQuota object, provides constraints that **limit aggregate resource consumption per namespace**. 
- It can limit the **quantity of objects** that can be created in a namespace **by type**, as well as the **total amount of compute resources** that may be consumed by resources in that project.

[![Image](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-kubernetes-service-namespaces-resource-quota.jpg "Azure AKS Kubernetes")](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-kubernetes-service-namespaces-resource-quota.jpg)

## Step-01: Create Namespace manifest
```yml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

## Step-02: Create ResourceQuota manifest
```yml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: ns-resource-quota
  namespace: dev
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi  
    pods: "5"    
    configmaps: "5" 
    persistentvolumeclaims: "5" 
    secrets: "5" 
    services: "5"                      
```


## Step-03: Create k8s objects & Test
```
# Create All Objects
kubectl apply -f Namespace-LimitRange-ResourceQuota.yml
kubectl apply -f kube-manifests/

# List Pods
kubectl get pods -n dev

# View Pod Specification (CPU & Memory)
kubectl get pod <pod-name> -o yaml -n dev

# Get & Describe Limits
kubectl get limits -n dev
kubectl describe limits default-cpu-mem-limit-range -n dev

# Get Resource Quota 
kubectl get quota -n dev
kubectl describe quota ns-resource-quota -n dev

# List Service
kubectl get svc -n dev

# Access Application
http://<Public-IP-from-List-Services-Output>/app1/index.html
```
## References:
- https://kubernetes.io/docs/tasks/administer-cluster/namespaces-walkthrough/
- https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/

## Additional References:
- https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-constraint-namespace/ 
- https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-constraint-namespace/