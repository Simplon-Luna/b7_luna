# **Commandes utilisées**

## 6.1

#### list services and types

```bash
kubectl get service 
```

#### list pods and types

```bash
kubectl get pods
```

#### describe running and failed pod

```bash
kubectl describe pods [name]
```

#### Create Azure Resource Group

```bash
az group create -l westus -n b7luna
```

#### Create AKS Cluster

```bash
az aks create -g b7luna -n AKSClusterLuna --enable-managed-identity --node-count 2 --enable-addons monitoring --enable-msi-auth-for-monitoring  --generate-ssh-keys
```

#### Create an AKS cluster with 4 nodes

```bash
az aks create -g b7luna -n KlusterLuna --enable-managed-identity --node-count 4 --enable-addons monitoring --enable-msi-auth-for-monitoring  --generate-ssh-keys
```

#### Connect to the cluster

```bash
az aks get-credentials --resource-group b7luna --name AKSClusterLuna
```

#### Deploy the application

[link](https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-cli#code-try-7)

voting.yml*

```bash
kubectl apply -f voting.yml
```

#### Create KT auth & pwd secret

```bash
kubectl create secret generic reddb-pass --from-literal=username=devuser --from-literal=password=password_redis_154
```

#### Volumes

We learned that Storage Account, PV creations and bindings aren't necessary, with the right settings, K8s is totally capable of handling it by itself and more reliably than human hand.

## 6.2

#### Connect to the cluster

```bash
az aks get-credentials --resource-group b7luna --name AKSClusterLuna
```

#### Add Gandi webhook jetstack with helm

[jetstack](https://github.com/bwolf/cert-manager-webhook-gandi)

```bash
helm repo add jetstack https://charts.jetstack.io
```

```bash
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --set installCRDs=true --version v1.10.1 --set 'extraArgs={--dns01-recursive-nameservers=8.8.8.8:53\,1.1.1.1:53}'
```

#### Add requirements for Ingress (controller) in K8s

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.4.0/deploy/static/provider/cloud/deploy.yaml
```

#### Gandi secret

```bash
kubectl create secret generic gandi-credentials --from-literal=api-token='xgUksnKHBnqmfatz1wES9Hi3'
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

#### To create an alias for a command on azure CLi 

alias [WhatWeWant]="[WhatIsChanged]"  
*Example :*  
```bash
alias k="kubectl"
```

#### To deploy resources with yaml file

kubectl apply -f [name-of-the-yaml-file]
*Example :*  
```bash
kubectl apply -f azure-vote.yaml
```

#### To check resources

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

#### To describe resources

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

#### To delete resources

```bash
kubectl delete deploy --all
kubectl delete svc --all
kubectl delete pvc --all
kubectl delete pv --all
az group delete --name [resourceGroupName] --yes --no-wait
```

#### To create a repository Helm and install Jetstack

*To create the repository and install Jetstack :*

```bash
helm repo add jetstack https://charts.jetstack.io
```

*To check the repository created and Jetstack version :*

```bash
helm search repo jetstack
```

#### To create a role for Gandi's secret and bind it to the webhook

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

#### To check TLS certificate in request order

```bash
kubectl get certificate
kubectl get certificaterequest
kubectl get order
kubectl get challenge
```

#### To describe TLS certificate in request order

```bash
kubectl describe certificate
kubectl describe certificaterequest
kubectl describe order
kubectl describe challenge
```

#### Get the IP address to point the DNS to nginx

```bash
k get ingress
```

#### Activate the autoscaler on an existing cluster

```bash
az aks update --resource-group b7luna --name AKSClusterd2 --enable-cluster-autoscaler --min-count 1 --max-count 8
```

#### To check the auto scaling creation

```bash
get HorizontalPodAutoscaler
```

*Example of how the results will display :*  

```bash
horizontalpodautoscaler.autoscaling/scaling-voteapp created
```

#### To check Webhook configuration

```bash
kubectl get ValidatingWebhookConfiguration -A
```

#### Delete Webhook configuration for a role

```bash
kubectl delete -A ValidatingWebhookConfiguration [rolename]  
```

*Example :*  

```bash
kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission
```

#### Pipeline scheduling

schedules:
- cron: "0 * * * *" # cron syntax defining the schedule - every hour at 0'
  displayName: Hourly main schedule # friendly name given to a specific schedule
  branches:
    include: # which branches the schedule applies to
    - main 
#    exclude: # which branches to exclude from the schedule
#    - [string]
  always: false # whether to always run the pipeline or only if there have been source code changes since the last successful scheduled run. The default is false.

#### Cron syntax

[Cron Syntax](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/scheduled-triggers?view=azure-devops&tabs=yaml#cron-syntax)

mm HH DD MM DW
 \  \  \  \  \__ Days of week
  \  \  \  \____ Months
   \  \  \______ Days
    \  \________ Hours
     \__________ Minutes
Field	Accepted values
Minutes	0 through 59
Hours	0 through 23
Days	1 through 31
Months	1 through 12, full English names, first three letters of English names
Days of week	0 through 6 (starting with Sunday), full English names, first three letters of English names
Values can be in the following formats.

Format	Example	Description
Wildcard	*	Matches all values for this field
Single value	5	Specifies a single value for this field
Comma delimited	3,5,6	Specifies multiple values for this field. Multiple formats can be combined, like 1,3-6
Ranges	1-3	The inclusive range of values for this field
Intervals	*/4 or 1-5/2	Intervals to match for this field, such as every fourth value or the range 1-5 with a step interval of 2
Example	Cron expression
Build every Monday, Wednesday, and Friday at 6:00 PM	0 18 * * Mon,Wed,Fri, 0 18 * * 1,3,5, or 0 18 * * 1-5/2
Build every 6 hours	0 0,6,12,18 * * *, 0 */6 * * * or 0 0-18/6 * * *
Build every 6 hours starting at 9:00 AM	0 9,15,21 * * * or 0 9-21/6 * * *

-----

I apologize for the confusion, you are correct that the kubectl create secret command with the --from-literal flag creates an opaque secret by default. In Kubernetes, when you create a secret using the --from-literal flag, the key-value pairs are encoded in base64.

To create a clear text secret, you can use the --dry-run flag, -o yaml flag and sed command to modify the yaml file, then create the secret using the modified yaml file.

Here's an example of how you can create a clear text secret:

Copy code
kubectl create secret generic my-secret --from-literal=version=1.0 --dry-run -o yaml | sed 's/opaque/string/' | kubectl apply -f -
This command creates a secret of type 'generic' named "my-secret" with a key "version" and the value "1.0" in plain text.

This command creates a new secret with the --dry-run flag and -o yaml flag which will output the yaml file representation of the secret, which is then piped to sed command which then replace the value opaque with string. This modified yaml is then passed to the kubectl apply command which creates the secret.

You can then use this command to update the secret as well.

It's important to note that storing sensitive information in clear text in Kubernetes secrets is not recommended as it could be exposed if the secret is compromised or accessed by an unauthorized user. You should use appropriate security measures to protect the secret and only use this approach if it's appropriate for your use case.