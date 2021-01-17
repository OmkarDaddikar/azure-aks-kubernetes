# Registering Domains to Azure DNS
## Step-01: Purchase Domain Name from Domain Registrar
  - A **Domain Registrar** is a company who can provide **Internet Domain Names**. 
    **Example:** GoDaddy, Wix, Namcheap, AWS Route53
  - They **verify** if the Internet domain you want to use is available and allow you to **purchase** it. 
  - Once the domain name is **registered**, you are the **legal owner** for the **domain name**. 
  - If you already have an Internet domain, you will use the current **domain registrar to delegate to Azure DNS**.
  - A domain is a **unique name** in the Domain Name System, for example **pilankarmasale.com**. 
  - A **DNS zone** is used to host the **DNS records** for a particular domain.
  - For example, the domain **pilankarmasale.com** may contain several DNS records such as 
mail.pilankarmasale.com (For a Mail Server) 
www.pilankarmasale.com (For a Website).
  - Azure DNS is **not the domain registrar**.
  - Azure DNS allows you to host a **DNS Zone** and manage the DNS records for a domain in Azure.
  - For **DNS queries** for a domain to reach Azure DNS, the domain must be **delegated to Azure DNS** from the parent domain.
[![Image](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-aks-delegate-domain-to-azure-dns.jpg "Azure AKS Kubernetes")](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-aks-delegate-domain-to-azure-dns.jpg)
## Step-02: Create DNS Zone
- Go to Service -> **DNS Zones**
- **Subscription:** Subscription Name (You need to have a paid subscription for this)
- **Resource Group:** dns-zones
- **Name:** pilankarmasale.com
- **Resource Group Location:** East US
- Click on **Review + Create**
## Step-03: Make a note of Azure Nameservers
- Go to Services -> **DNS Zones** -> **pilankarmasale.com**
- Make a note of Nameservers
```
ns1-04.azure-dns.com.
ns2-04.azure-dns.net.
ns3-04.azure-dns.org.
ns4-04.azure-dns.info.
```
## Step-04: Update Nameservers at your Domain Provider
- **Verify before updation**
```
nslookup -type=SOA pilankarmasale.com
nslookup -type=NS pilankarmasale.com
```
- Go to AWS Route53
- Go to Services -> Route53 -> Registered Domains -> pilankarmasale.com
- Click on **Add or edit name servers**
- Update Azure Name servers here and click on **Update**
- Click on **Hosted Zones**
- Delete the hosted zone with name **pilankarmasale.com**
- **Verify after updation**
```
nslookup -type=SOA pilankarmasale.com 8.8.8.8
nslookup -type=NS pilankarmasale.com 8.8.8.8
```
## Kubernetes ExternalDNS to create Record Sets in Azure DNS from AKS
- ExternalDNS **synchronizes** exposed Kubernetes Services and Ingresses with **DNS providers**.
- In simple terms, ExternalDNS allows you to **control DNS records dynamically via Kubernetes resources** in a DNS provider-agnostic way.
- **Reference:** https://github.com/kubernetes-sigs/external-dns 
[![Image](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-aks-ingress-external-dns.jpg "Azure AKS Kubernetes")](https://github.com/OmkarDaddikar/azure-aks-kubernetes/blob/master/Images/azure-aks-ingress-external-dns.jpg)

## Step-05: Create External DNS Manifests
- External-DNS needs permissions to Azure DNS to modify (Add, Update, Delete DNS Record Sets)
- We can provide permissions to External-DNS pod in two ways in Azure 
  - Using Azure Service Principal
  - Using Azure Managed Service Identity (MSI)
- We are going to use `MSI` for providing necessary permissions here which is latest and greatest in Azure as on today.
### Gather Information Required for azure.json file
```
# To get Azure Tenant ID
az account show --query "tenantId"

# To get Azure Subscription ID
az account show --query "id"
```

### Create azure.json file
```json
{
  "tenantId": "c81f465b-99f9-42d3-a169-8082d61c677a",
  "subscriptionId": "82808767-144c-4c66-a320-b30791668b0a",
  "resourceGroup": "dns-zones", 
  "useManagedIdentityExtension": true,
  "userAssignedIdentityID": "404b0cc1-ba04-4933-bcea-7d002d184436"  
}
```

### Review external-dns.yml manifest
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: external-dns
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: external-dns
rules:
- apiGroups: [""]
  resources: ["services","endpoints","pods"]
  verbs: ["get","watch","list"]
- apiGroups: ["extensions","networking.k8s.io"]
  resources: ["ingresses"] 
  verbs: ["get","watch","list"]
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list"]
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: external-dns-viewer
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: external-dns
subjects:
- kind: ServiceAccount
  name: external-dns
  namespace: default
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: external-dns
spec:
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: external-dns
  template:
    metadata:
      labels:
        app: external-dns
    spec:
      serviceAccountName: external-dns
      containers:
      - name: external-dns
        image: registry.opensource.zalan.do/teapot/external-dns:latest
        args:
        - --source=service
        - --source=ingress
        #- --domain-filter=example.com # (optional) limit to only example.com domains; change to match the zone created above.
        - --provider=azure
        #- --azure-resource-group=externaldns # (optional) use the DNS zones from the specific resource group
        volumeMounts:
        - name: azure-config-file
          mountPath: /etc/kubernetes
          readOnly: true
      volumes:
      - name: azure-config-file
        secret:
          secretName: azure-config-file
```
## Step-06: Create MSI - Managed Service Identity for External DNS to access Azure DNS Zones
### Create Manged Service Identity (MSI)
- Go to **All Services** -> **Managed Identities** -> **Add**
- **Resource Name:** aks-prod-externaldns-access-to-dnszones
- **Subscription:** Pay-as-you-go
- **Resource Group:** aks-prod
- **Location:** Central US
- Click on **Create**

### Add Azure Role Assignment in MSI
- **Open MSI** -> **aks-prod-externaldns-access-to-dnszones**
- Click on **Azure Role Assignments** -> **Add role assignment**
- **Scope:** Resource Group
- **Subscription:** Pay-as-you-go
- **Resource Group:** dns-zones
- **Role:** Contributor

### Make a note of Client Id and update in azure.json
- Go to **Overview** -> Make a note of **Client ID**
- Update in **azure.json** value for **userAssignedIdentityID**
```
  "userAssignedIdentityID": "de836e14-b1ba-467b-aec2-93f31c027ab7"
```

## Step-07: Associate MSI in AKS Cluster VMSS
- Go to **All Services** -> **Virtual Machine Scale Sets (VMSS)** -> Open **aks-prod** related VMSS (aks-agentpool-27193923-vmss)
- Go to **Settings** -> **Identity** -> **User Assigned** -> **Add** -> aks-prod-externaldns-access-to-dnszones

## Step-08: Create Kubernetes Secret and Deploy ExternalDNS
```
# Create Secret
kubectl create secret generic azure-config-file --from-file=azure.json

# List Secrets
kubectl get secrets

# Deploy ExternalDNS 
kubectl apply -f External-DNS-Deploy.yml

# Verify ExternalDNS Logs
kubectl logs -f $(kubectl get po | egrep -o 'external-dns[A-Za-z0-9-]+')
```

```Logs
# Error Type: 400
time="2020-08-24T11:25:04Z" level=error msg="azure.BearerAuthorizer#WithAuthorization: Failed to refresh the Token for request to https://management.azure.com/subscriptions/82808767-144c-4c66-a320-b30791668b0a/resourceGroups/dns-zones/providers/Microsoft.Network/dnsZones?api-version=2018-05-01: StatusCode=400 -- Original Error: adal: Refresh request failed. Status Code = '400'. Response body: {\"error\":\"invalid_request\",\"error_description\":\"Identity not found\"}"

# Error Type: 403
Notes: Error 403 will come when our Managed Service Identity dont have access to respective destination resource 

# When all good, we should get log as below
time="2020-08-24T11:27:59Z" level=info msg="Resolving to user assigned identity, client id is 404b0cc1-ba04-4933-bcea-7d002d184436."
```
## Step-09: Deploy Application and Test
### Deploy Application
```
# Deploy Application
kubectl apply -R -f kube-manifests/

# Verify Pods and Services
kubectl get po,svc

# Verify Ingress
kubectl get ingress
```

### Verify logs in External DNS Pod
- Wait for 3 to 5 minutes for Record Set update in DNZ Zones
```
# Verify ExternalDNS Logs
kubectl logs -f $(kubectl get po | egrep -o 'external-dns[A-Za-z0-9-]+')
```
- External DNS Pod Logs
```log
time="2020-08-24T11:30:54Z" level=info msg="Updating A record named 'app1' to '20.37.141.33' for Azure DNS zone 'pilankarmasale.com'."
time="2020-08-24T11:30:55Z" level=info msg="Updating TXT record named 'app1' to '\"heritage=external-dns,external-dns/owner=default,external-dns/resource=ingress/default/nginxapp1-ingress-service\"' for Azure DNS zone 'pilankarmasale.com'."
```

### Verify Record Set in DNZ Zones -> pilankarmasale.com
- Go to **All Services** -> **DNS Zones** -> **pilankarmasale.com**
- Verify if we have `www.pilankarmasale.com` & `app1.pilankarmasale.com` created
```
# Template Command
az network dns record-set a list -g <Resource-Group-dnz-zones> -z <yourdomain.com>

# Replace DNS Zones Resource Group and yourdomain
az network dns record-set a list -g dns-zones -z pilankarmasale.com
```
- Perform `nslookup` test
```
# nslookup Test
nslookup app1.pilankarmasale.com
Server:		192.168.0.1
Address:	192.168.0.1#53

Non-authoritative answer:
Name:	app1.pilankarmasale.com
Address: 20.37.141.33
```

### Access Application and Test
```
# Access Application
http://www.pilankarmasale.com

# Access User Management Web Application
http://app1.pilankarmasale.com
Username: admin101
Password: password101
```
## Ingress Annotation Reference
- https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/

## External DNS References
- https://github.com/kubernetes-sigs/external-dns
- https://github.com/kubernetes-sigs/external-dns/blob/master/docs/faq.md

## Other References
- https://docs.nginx.com/nginx-ingress-controller/
- https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/azure.md
- Open Issue and Break fix: https://github.com/kubernetes-sigs/external-dns/issues/1548
- https://github.com/kubernetes/ingress-nginx/tree/master/charts/ingress-nginx#configuration
- https://github.com/kubernetes/ingress-nginx/blob/master/charts/ingress-nginx/values.yaml
- https://kubernetes.github.io/ingress-nginx/
