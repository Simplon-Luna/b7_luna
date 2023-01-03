## Partie 1

# **Commandes utilisées**


### list services
```bash
kubectl get service 
```
### list pods
```bash
kubectl get pods
```
### describe running and failed pod
```bash
kubectl describe pods [name]
```

### Create Azure Resource Group
```bash
az group create -l westus -n b7luna
```

### Create AKS Cluster
```bash
az aks create -g b7luna -n AKSClusterLuna --enable-managed-identity --node-count 2 --enable-addons monitoring --enable-msi-auth-for-monitoring  --generate-ssh-keys
```
### Connect to the cluster
```bash
az aks get-credentials --resource-group b7luna --name AKSClusterLuna
```
### Deploy the application
[link](https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-cli#code-try-7)

voting.yml
```bash
kubectl apply -f voting.yml
```
### Determine the networking service type
```bash
kubectl get service votingapp-azure --watch
```
### Create KT auth & pwd secret
```bash
kubectl create secret generic reddb-pass --from-file=./username.txt --from-file=./password.txt
```
```bash
kubectl create secret generic reddb-pass --from-literal=username=devuser --from-literal=password=password_redis_154
```
## Volumes

links :

[Multiple Nodes](https://stackoverflow.com/questions/54845025/does-kubernetes-support-persistent-volumes-shared-between-multiple-nodes-in-a-cl)

[AKS Multiple Nodes](https://learn.microsoft.com/fr-fr/azure/aks/azure-files-volume)

[AKS Storage](https://learn.microsoft.com/en-us/azure/aks/concepts-storage)

[AKS Storage Driver](https://learn.microsoft.com/en-us/azure/aks/csi-storage-drivers)

[AKS Azure file claim](https://learn.microsoft.com/en-us/azure/aks/azure-files-volume#mount-file-share-as-an-persistent-volume)

[Create PV](https://learn.microsoft.com/en-us/azure/aks/azure-files-volume)

#### Create KT secret for access to file share
```bash
kubectl create secret generic azure-secret --from-literal=azurestorageaccountname=b7lstorage --from-literal=azurestorageaccountkey=7TK5/DzbGowqfBKhCbwJR9c3JA41d+J0TTpbFEWFRXDyerLXu96xNAauAK7M+FiNAj4bAlHTdvrj+AStWF805g==
```
## Partie 2

#### Creation Kluster avec ACR
##### set this to the name of your Azure Container Registry.  It must be globally unique
```bash
MYACR=lunacr
```
##### Run the following line to create an Azure Container Registry if you do not already have one
```bash
az acr create -n $MYACR -g b7luna --sku basic
```
##### Create an AKS cluster with ACR integration
```bash
az aks create -g b7luna -n KlusterLuna --enable-managed-identity --node-count 4 --enable-addons monitoring --enable-msi-auth-for-monitoring  --generate-ssh-keys --attach-acr $MYACR
```
### Connect to the cluster
```bash
az aks get-credentials --resource-group b7luna --name KlusterLuna
```
### Add Gandi webhook jetstack with helm

[jetstack](https://github.com/bwolf/cert-manager-webhook-gandi)
```bash
helm repo add jetstack https://charts.jetstack.io
```

```bash
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --set installCRDs=true --version v1.9.1 --set 'extraArgs={--dns01-recursive-nameservers=8.8.8.8:53\,1.1.1.1:53}'
```
#### Gandi secret
```bash
kubectl create secret generic gandi-credentials --namespace cert-manager --from-literal=api-token='2DqJpnKJljl9yWQIolq2xRXO'
```

#### install cert-manager webhook for gandi
```bash
helm install cert-manager-webhook-gandi --repo https://bwolf.github.io/cert-manager-webhook-gandi --version v0.2.0 --namespace cert-manager --set features.apiPriorityAndFairness=true  --set logLevel=6 --generate-name
```
#### create secret role and bind for webhook
```bash
kubectl create role access-secret --verb=get,list,watch,update,create --resource=secrets
```
```bash
kubectl create rolebinding --role=access-secret default-to-secrets --serviceaccount=cert-manager:cert-manager-webhook-gandi-1665664967
```
Apply ingress -> issuer -> certificate

#### update AKS with autoscale
```bash
az aks update --resource-group b7luna --name KlusterLuna --enable-cluster-autoscaler --min-count 1 --max-count 8
```
#### Autoscaling

[Autoscaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)

[Autoscaling Walkthrough](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)

----

## To create an alias for a command on azure CLi 
alias [WhatWeWant]="[WhatIsChanged]"  
*Example :*  
```bash
alias k="kubectl"
```

## To deploy resources with yaml file
kubectl apply -f [name-of-the-yaml-file]
*Example :*  
```bash
kubectl apply -f azure-vote.yaml
```

## To check resources
```bash
kubectl get nodes
kubectl get pods
kubectl get services
kubectl get deployments
kubectl get events
kubectl get secrets
kubectl get logs
```
*To keep verifying the resources add --watch at the end of the command :*
*Example :*
```bash
kubectl get services --watch
```
*To check the resources according to their namespace, add --namespace after the command and the namespace's name :*
*Example :*
```bash
kubectl get services --namespace [namespace's-name]
```

## To describe resources
```bash
kubectl describe nodes
kubectl describe pods
kubectl describe services # or svc
kubectl describe deployment # or deploy
kubectl describe events
kubectl describe secrets
kubectl describe logs
```
*To specify which resource needs to be described just put the resource ID at the end of the command.*
*Example :*
```bash
kubectl describe svc redis-service
```
*To access to all the logs from all containers :*
```bash
kubectl logs podname --all-containers
```
*To access to the logs from a specific container :*
```bash
kubectl logs podname -c [container's-name]
```
*To list all events from a specific pod :*
```bash
kubectl get events --field-selector [involvedObject].name=[podsname]
```

## To delete resources
```bash
kubectl delete deploy --all
kubectl delete svc --all
kubectl delete pvc --all
kubectl delete pv --all
az group delete --name [resourceGroupName] --yes --no-wait
```

## To create a repository Helm and install Jetstack
*To create the repository and install Jetstack :*
```bash
helm repo add jetstack https://charts.jetstack.io
```
*To check the repository created and Jetstack version :*
```bash
helm search repo jetstack
```

## To create a role for Gandi's secret and bind it to the webhook
*To create the role :*
```bash
kubectl create role [role-name] --verb=[Authorised-actions] --resource=[Authorised-resource]
```
*Example :*  
```bash
kubectl create role access-secrets --verb=get,list,watch,update,create --resource=secrets
```
*To bind it :*  
```bash
kubectl create rolebinding --role=[role-name] [role-name] --serviceaccount=[group]:[group-item]
```
*Example :*  
```bash
kubectl create rolebinding --role=access-secrets default-to-secrets --serviceaccount=cert-manager:cert-manager-webhook-gandi-1665665029
```

## To check TLS certificate in request order
```bash
kubectl get certificate
kubectl get certificaterequest
kubectl get order
kubectl get challenge
```

## To describe TLS certificate in request order
```bash
kubectl describe certificate
kubectl describe certificaterequest
kubectl describe order
kubectl describe challenge
```

## Get the IP address to point the DNS to nginx
```bash
k get ingress
```

## Activate the autoscaler on an existing cluster
```bash
az aks update --resource-group b7luna --name AKSClusterd2 --enable-cluster-autoscaler --min-count 1 --max-count 8
```

## To check the auto scaling creation
```bash
get HorizontalPodAutoscaler
```
*Example of how the results will display :*  
```bash
horizontalpodautoscaler.autoscaling/scaling-voteapp created
```

## To check Webhook configuration
```bash
kubectl get ValidatingWebhookConfiguration -A
```

## Delete Webhook configuration for a role
```bash
kubectl delete -A ValidatingWebhookConfiguration [rolename]  
```
*Example :*  
```bash
kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission
```