# Kubernetes Namespaces - LimitRange - Declarative using YAML

[![Image](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-kubernetes-service-namespaces-3.jpg "Azure AKS Kubernetes")](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-kubernetes-service-namespaces-3.jpg)

[![Image](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-kubernetes-service-RBAC-CR-CRB-1.jpg "Azure AKS Kubernetes")](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-kubernetes-service-RBAC-CR-CRB-1.jpg)

## Step-01: Introduction
- By default, containers run with **unbounded compute resources**on a Kubernetes cluster.
- With resource quotas, **cluster administrators can restrict** resource consumption and creation on a namespace basis. 
- Within a namespace, a Pod or Container can consume as much CPU and memory as defined by the **namespace's resource quota**. 
- There is a **concern that** one Pod or Container could monopolize all available resources. 
- A **LimitRange is a policy** to constrain resource allocations (to Pods or Containers) in a namespace.

A **LimitRange** provides constraints that can:
- Enforce **minimum and maximum compute resources** usage per Pod or Container in a namespace.
- Enforce **minimum and maximum storage request per PersistentVolumeClaim** in a namespace.
- Enforce a **ratio between request and limit for a resource** in a namespace.
- Set **default request/limit for compute resources in a namespace** and automatically **inject** them to Containers at runtime.

[![Image](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-kubernetes-service-namespaces-limit-range.jpg "Azure AKS Kubernetes")](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-kubernetes-service-namespaces-limit-range.jpg)

## Step-01: Create Namespace manifest
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

## Step-02: Create LimitRange manifest
- Instead of specifying `resources like cpu and memory` in every container spec of a pod defintion, we can provide the default CPU & Memory for all containers in a namespace using `LimitRange`
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: ns-resource-quota
  namespace: dev
spec:
  limits:
    - default:
        memory: "512Mi" # If not specified the Container's memory limit is set to 512Mi, which is the default memory limit for the namespace.
        cpu: "500m"     # If not specified default limit is 1 vCPU per container 
      defaultRequest:
        memory: "256Mi" # If not specified default it will take from whatever specified in limits.default.memory
        cpu: "300m"     # If not specified default it will take from whatever specified in limits.default.cpu
      type: Container                        
```

## Step-03: Update all k8s manifest with namespace
- Update all files from with `namespace: dev` in top metadata section in folder `kube-manifests/` 
- **Example**
```yaml
# Deployment Manifest metadata section
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1-nginx-deployment
  labels:
    app: app1-nginx
  namespace: dev    # Added namespace
spec:

# Service Manifest metadata section
apiVersion: v1
kind: Service
metadata:
  name: app1-nginx-clusterip-service
  labels:
    app: app1-nginx
  namespace: dev   # Added namespace
spec: 
```

## Step-04: Create k8s objects & Test
```
# Create All Objects
kubectl apply -f Namespace-LimitRange.yml
kubectl apply -f kube-manifests/

# List Pods
kubectl get pods -n dev 

# View Pod Specification (CPU & Memory)
kubectl get pod <pod-name> -o yaml -n dev

# Get & Describe Limits
kubectl get limits -n dev
kubectl describe limits default-cpu-mem-limit-range -n dev

# List Services
kubectl get svc -n dev

# Access Application
http://<Public-IP-from-List-Services-Output>/app1/index.html
```

## References:
- https://kubernetes.io/docs/tasks/administer-cluster/namespaces-walkthrough/
- https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/
- https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/