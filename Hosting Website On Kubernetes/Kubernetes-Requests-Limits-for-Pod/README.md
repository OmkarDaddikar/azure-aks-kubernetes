# Kubernetes - Requests and Limits for Pods

## Step-01: Introduction
- We can specify how much each container in a pod needs the resources like CPU & Memory. 
- When we provide this information in our pod, the scheduler uses this information to decide which node to place the Pod on based on availability of k8s worker Node CPU and Memory Resources. 
- When you specify a resource limit for a Container, the kubelet enforces those `limits` so that the running container is not allowed to use more of that resource than the limit you set. 
-  The kubelet also reserves at least the `request` amount of that system resource specifically for that container to use.

[![Image](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-kubernetes-service-resources-requests-limits-1.jpg "Azure AKS Kubernetes")](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-kubernetes-service-resources-requests-limits-1.jpg)

[![Image](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-kubernetes-service-resources-requests-limits-2.jpg "Azure AKS Kubernetes")](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-kubernetes-service-resources-requests-limits-2.jpg)

## Step-02: Add Requests & Limits & Review k8s Manifests
- **Folder:** kube-manifests
```yaml
          # Requests & Limits    
          resources:
            requests:
              cpu: "100m" 
              memory: "128Mi"
            limits:
              cpu: "200m"
              memory: "256Mi"                                                         
```

## Step-03: Create k8s objects & Test
```
# Create All Objects
kubectl apply -f kube-manifests/

# List Pods
kubectl get pods

# List Services
kubectl get svc

# Access Application 
http://<Public-IP-from-List-Services-Output>/app1/index.html
```

## References:
- https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/