# Implement SSL Using Lets Encrypt
[![Image](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-aks-ingress-ssl-letsencrypt.jpg "Azure AKS Kubernetes")](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-aks-ingress-ssl-letsencrypt.jpg)
## Step-01: Install Cert Manager
```
# Install the CustomResourceDefinition resources separately
kubectl apply --validate=false -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.13/deploy/manifests/00-crds.yaml

# Label the ingress-basic namespace to disable resource validation
kubectl label namespace ingress-basic cert-manager.io/disable-validation=true

# Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io

# Update your local Helm chart repository cache
helm repo update

# Install the cert-manager Helm chart
helm install \
  cert-manager \
  --namespace ingress-basic \
  --version v0.13.0 \
  jetstack/cert-manager

# Verify Cert Manager pods
kubectl get pods --namespace ingress-basic
```

## Step-02: Review or Create Cluster Issuer Kubernetes Manifest
### Review Cluster Issuer Kubernetes Manifest
- Create or Review Cert Manager Cluster Issuer Kubernetes Manigest
```yml
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: omkar.dd94@gmail.com
    privateKeySecretRef:
      name: letsencrypt
    solvers:
      - http01:
          ingress:
            class: nginx
```
### Deploy Cluster Issuer
```
# Deploy Cluster Issuer
kubectl apply -f CertManager-Cluster-Issuer.yml
```
## Step-03: Create and Review Application & Ingress SSL Kubernetes Manifests
- Deployment-and-ClusterIP-Service.yml
- 01-Persistent-Volume-Claim.yml
- 02-User-Management-ConfigMap.yml
- 03-MySQL-Deployment-and-ClusterIP-Service.yml
- 04-User-Management-Deployment-and-ClusterIP-Service.yml
- 05-Kubernetes-Secrets.yml
- Ingress-SSL.yml

## Step-04: Deploy All Manifests & Verify
- Certificate Request, Generation, Approval and Download and be ready might take from 1 hour to couple of days if we make any mistakes and also fail.
```
# Deploy
kubectl apply -R -f kube-manifests/

# Verify Pods
kubectl get pods

# Verify Cert Manager Pod Logs
kubectl get pods -n ingress-basic
kubectl  logs -f <cert-manager-55d65894c7-sx62f> -n ingress-basic #Replace Pod name

# Verify SSL Certificates (It should turn to True)
kubectl get certificate
```
```log
NAME                         READY   SECRET                       AGE
pilankarmasale-secret        True    pilankarmasale-secret        45m
app1-pilankarmasale-secret   True    app1-pilankarmasale-secret   45m
```
```
# Sample Success Log
I0824 13:09:00.495721       1 controller.go:129] cert-manager/controller/orders "msg"="syncing item" "key"="default/pilankarmasale-secret-2792049964-67728538" 
I0824 13:09:00.495900       1 sync.go:102] cert-manager/controller/orders "msg"="Order has already been completed, cleaning up any owned Challenge resources" "resource_kind"="Order" "resource_name"="app2-kubeoncloud-secret-2792049964-67728538" "resource_namespace"="default" 
I0824 13:09:00.496904       1 controller.go:135] cert-manager/controller/orders "msg"="finished processing work item" "key"="default/app1-pilankarmasale-secret-2792049964-67728538
```
## Step-05: Access Application
```
https://www.pilankarmasale.com
http://app1.pilankarmasale.com
```
## Cert Manager
- https://docs.cert-manager.io/en/latest/reference/issuers.html
- https://docs.cert-manager.io/en/latest/reference/ingress-shim.html
- https://cert-manager.io/docs/release-notes/
- https://cert-manager.io/docs/reference/api-docs/#cert-manager.io/v1beta1.Issuer
- https://letsencrypt.org/how-it-works/