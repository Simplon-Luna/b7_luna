<div style='text-align: justify;'>

<div id='top'/>

### Summary

###### [00 - Daily Scrum](#Scrum)

###### [01 - Kubernetes, AKS and Azure Pipelines doc reading](#Doc)

###### [02 - Architecture Topology](#Topology)

###### [03 - Resource List](#Resources)

###### [04 - Creation of a resource group](#RG)

###### [05 - Creation of a storage account (standard GRS)](#Storacc)

###### [06 - Creation of the AKS Cluster (with SSH keys generated)](#AKS)

###### [07 - Connecting the AKS Cluster and Azure](#Connecting)

###### [08 - Creation of the redis secret](#RedSecret)

###### [09 - Connecting to Azure DevOps Pipelines](#DevOps)

###### [10 - Creation of a test pipeline](#Pipeline)

###### [11 - Checks and tests](#Checks)

###### [12 - Error messages](#Error)

###### [13 - Trying to find a solution](#Solution)

###### [14 - Updating the voting app on the script](#Updating)

###### [15 - Remove the PV](#PV)

###### [16 - Delete everything and start again](#Again)

###### [17 - Add Ingress](#Ingress)

###### [18 - Install Cert-manager and Jetstack (for gandi)](#Cert-manager)

###### [19 - Creation of a Gandi secret](#Gsecret)

###### [20 - Install cert-manager webhook for gandi](#Webhook)

###### [21 - Creation of secret role and binding for webhook](#Binding)

###### [22 - Check the certificate](#Certificate)

###### [23 - Scheduling](#Scheduling)

###### [24 - Pipeline debugging](#Debugging)

###### [25 - How to get a certificate Summary](#Summary)

###### [26 - Executive summary](#ExecSummary)

###### [27 - Technical Architecture Document of deployed infrastructure](#DAT)

###### [28 - Check consumption](#Consumption)

###### [29 - Usefull Commands](#UsefullCommands)

<div id='Scrum'/>  

### **Scrum quotidien**

Scrum Master = Me, myself and I
Daily personnal reactions with reports and designations of first tasks for the day.

Frequent meeting with other coworkers to study solutions to encountered problems together.

[scrums](https://github.com/simplon-lerouxDunvael/Brief_7/blob/main/Plans_et_demarches/Scrum.md)

[&#8679;](#top)

<div id='Docs'/>  

#### **Kubernetes, AKS and Azure Pipelines doc reading**

Lecture des documentations afin de déterminer les fonctionnements, prérequis et outils/logiciels nécessaires pour remplir les différentes tâches du Brief 7.

[&#8679;](#top)  

<div id='Topology'/>  

#### **Architecture Topology**

Infrastructure Plannifiée

![application_function](https://user-images.githubusercontent.com/108001918/215785565-1c0a7fac-5c4d-46fb-8f0f-070392580336.png)

![pipeline_process](https://user-images.githubusercontent.com/108001918/215771546-dd5bb6bd-c13e-41b7-992f-ea0a3c0b75d8.png)

[&#8679;](#top)  

<div id='Ressources'/>  

#### **Resource List**

-----------
| Ressources | Cluster AKS | Redis |  Voting App |
| :--------: | :--------: | :--------: | :--------: |
| Azure service | ✓ | ✓ | ✓ |
| Azure DevOps | ✓ | ✓ | ✓ |
| resource group | ✓ |✓ | ✓ |
| SSH (port) | N/A | 6379 | 80 |
| CPU | N/A | 100m-250m | 100m-250m |
| Memory | N/A | 128mi-256mi | 128mi-256mi |
| Image | N/A | redis:latest  | simplonasa/azure_voting_app:v1.0.11 |
| Load Balancer | N/A | ✓ puis ✗ | ✓ |
| ClusterIP | N/A | ✗ puis ✓ | ✗ |
| Kubernetes secret | ✓ | ✓ | ✓ |
| Persistent Vol. Claim (3Gi)| N/A | ✓ | ✗ |
| Ingress | ✓ | ✗ | ✓ |
| Nginx | ✓ | ✗ | ✗ |
| DNS | ✓ | N/A | ✓ |
| Cert-manager | N/A | N/A | v1.10.1 |
| Certificat TLS | N/A | N/A | ✓ |

ID Subscription :
a1f74e2d-ec58-4f9a-a112-088e3469febb

[&#8679;](#top)  

<div id='RG'/>  

### **Creation of a resource group**

```bash
az group create --location francecentral --name b7duna
```

[&#8679;](#top)

<div id='Storacc'/>  

### **Creation of a storage account (standard GRS)**

I learned that Storage Account, PV creations and bindings aren't necessary, with the right settings, Kubernetes is totally capable of handling it by itself and more reliably than human hand.

[&#8679;](#top)

<div id='AKS'/>  

### **Creation of the AKS Cluster (with SSH keys generated)**

```bash
az aks create -g b7duna -n AKSClusterDuna --enable-managed-identity --node-count 2 --enable-addons monitoring --enable-msi-auth-for-monitoring  --generate-ssh-keys
```

[&#8679;](#top)

<div id='Connecting'/>  

### **Connecting the AKS Cluster and Azure**

```bash
az aks get-credentials --resource-group b7duna --name AKSClusterDuna
```

[&#8679;](#top)

<div id='RedSecret'/>  

### **Creation of the redis secret**

```bash
kubectl create secret generic redis-secret-duna --from-literal=username=devuser --from-literal=password=password_redis_154
```

[&#8679;](#top)

<div id='DevOps'/>  

### **Connecting to Azure DevOps Pipelines**

First I went to Azure DevOps, created a project and clicked on Pipelines : <https://dev.azure.com/dlerouxext/b7duna/_build>  
Then I added a service connection (Project settings > service connections > add > kubernetes).

![service_connection2](https://user-images.githubusercontent.com/108001918/210520992-0536c68a-17b6-4b8a-91e4-2bccb2159e75.png)

[&#8679;](#top)

<div id='Pipeline'/>  

### **Creation of a test pipeline**

I created a new pipeline and provided its location (Github.yaml) and the repository (Brief_7). Then I chose a starter template and used the assistant to add a Kubectl task.  
Finally I saved it and ran it.

![pipeline_job_run](https://user-images.githubusercontent.com/108001918/210521068-ec3cc98c-e2ab-46a7-9d46-3cadd39a3c37.png)

[&#8679;](#top)

<div id='Checks'/>  

### **Checks and tests**

In order to understand how the pipeline works and the path used on the pipeline's environment vm, I used the commands ```pwd``` and ```ls -la``` on my pipeline script :

![pipeline_job_run2](https://user-images.githubusercontent.com/108001918/210543908-3f4670ec-8fdb-444f-acb7-e728a63d2d48.png)

It allowed me to know which path I needed to put to refer the .yaml file to use in the pipeline.

[&#8679;](#top)

<div id='Error'/>  

### **Error messages**

I received an error at the end of the job. It seems that Azure does not have the rights to create a persistent volume and the PV claim.

![Error_pipeline](https://user-images.githubusercontent.com/108001918/210544375-1f1e042e-a659-4c1f-941b-49a9ce07d471.png)

On the Azure CLI I searched the service account default used by Azure to run the pipeline with the following command :

```bash
kubectl get serviceaccounts/default
```

Alfred tried to bind an admin role he created to the Azure service account Default to see if we could get admin rights on the Kubernetes cluster. Sadly it did not work.

![Error_pipeline2](https://user-images.githubusercontent.com/108001918/210545569-b0ce0e74-e461-4407-b3fc-69c6ecfbaad5.png)

[&#8679;](#top)

<div id='Solution'/>  

### **Trying to find a solution**

I recreated my Kubernetes Service Connection and checked "Use cluster admin credentials". As Alfred changed the credentials previously, when I reran my pipeline I had no rights issue.

Then I focused on the PV and PVC issue.

I created a container in my storage account in order to be able to use my fileshare for the PV and PVC.

My PV displayed but was not mounted thus my redis container could not be created.

To understand the errors I had I checked the events :

```bash
kubectl get events
```

```bash
kubectl get events --sort-by='.metadata.creationTimestamp'
```

```bash
kubectl get events --sort-by='.metadata.creationTimestamp' -w
```

[&#8679;](#top)

<div id='Scrum'/>  

### **Updating the voting app on the script**

I changed the previous version of the voting app with the new one : simplonasa/azure_voting_app:v1.0.11 and my container for the Voting app was successfully created.
It then displayed in CrashLoopBackOff because redis was not created but now I just needed to find a solution for redis and the PV/PVC for everything to work properly.

[&#8679;](#top)

<div id='Updating'/>  

### **Creation of a storage share for the storage account**

I checked if I had a storage share for my storage account with the command :

```bash
az storage share list --account-name b7dstoracc --account-key Ha/rrRrMwoLotpOK1wT5a1dphjPgfa0z9NZjf7W/1veO6nhHgNtzvjFyIK+y1oBy+I92/y73CPVp+AStu1jQQQ==
```

I did not, so I created a storage share directly on my storage account.

```bash
az storage share create --account-name b7dstoracc --name b7d-redis-fileshare --account-key Ha/rrRrMwoLotpOK1wT5a1dphjPgfa0z9NZjf7W/1veO6nhHgNtzvjFyIK+y1oBy+I92/y73CPVp+AStu1jQQQ==
```

Then I verified that it had been successfully created.

![storage-share_check](https://user-images.githubusercontent.com/108001918/210567347-d9933eb0-4cf3-4753-a70e-1ed4170ecbf9.png)

Then I check my pods and services :
kubectl get pods
kubectl get services

As everything was running perfectly, I used the IP address to connect to the Voting App. It worked fine. Then I deleted the redis pod ```kubectl delete pod redis-service``` and typed ```kubcetl get pods```. The redis service was automatically renewed.  
Finally, I refreshed the voting app page and found out that the votes count had not been reset. All the containers were working and the persistent volume as well.

[&#8679;](#top)

<div id='PV'/>  

### **Remove the PV**

As Kubernetes automatically creates a PV when a PVC is created, I removed the PV from my script and decided to start again without creating the storage account, the storage share to verify if Kubernetes would do everything automatically. Then I ran the pipeline.

When I searched for the PVC and the pods with kubectl commands on Azure CLI, their status showed that it did not work properly.

![not_working_AGAIN](https://user-images.githubusercontent.com/108001918/210743380-128d1882-c8ad-45f6-a2c4-4f159585c20e.png)

After some researches *(and screechs)*, I modified the ```volumes``` part on the redis container with the persistentVolumeClaim and removed the references to the PV. I updated it as well in the ```volumeMounts``` part. Finally, I relaunched the pipeline and the job ran successfully.

[&#8679;](#top)

<div id='Again'/>  

### **Delete everything and start again**

I decided to delete my resource groups to try once again from scratch and check if it also worked when Alfred and Bryan weren't watching *(just in case the code gets pressured to work when they are present, we never know)*.

![pipeline_working](https://user-images.githubusercontent.com/108001918/210748762-a67a9983-bf18-480d-af1b-de5107fcc2b8.png)

![voting-app_working](https://user-images.githubusercontent.com/108001918/210749443-339f0a3d-befc-4b02-bce9-21c532321b29.png)

![working](https://user-images.githubusercontent.com/108001918/210749947-e702d1aa-9dfe-4591-a5f0-fdbaef5b6e51.png)

![persistent_working](https://user-images.githubusercontent.com/108001918/210750285-4f2fea62-585e-41d3-89f7-d9529023eec3.png)

[&#8679;](#top)

<div id='Ingress'/>  

### **Add Ingress**

As the scheduling was working, I decided to install Ingress to be able to access to the voting app from an url http I chose (and that i would later link to my dns record).  

I removed the Load balancer in my azure-vote.yaml file in order to put ClusterIP so that ingress can provide an IP address to use for my DNS record.  

![clusterIP_for_ingress](https://user-images.githubusercontent.com/108001918/210793328-a1052a6d-c03c-4dc7-bc24-0331170b7aac.png)

![no_tls](https://user-images.githubusercontent.com/108001918/210793423-5cf4b0af-93ea-4a51-b412-6439adef0f04.png)

I removed the TLS parts, the host and the TLS annotations in my ingress.yaml file. Then I applied it ```kubectl apply -f ingress.yaml``` and checked it with ```kubectl get ingress```.

But I had no IP address. So I installed everything that was necessary for ingress with the following command :

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.4.0/deploy/static/provider/cloud/deploy.yaml
```

After this step, ingress finally had an IP address. Then I created a "A" DNS record with the ingress IP address. I checked ingress and it now displayed with : smoothie.simplon-duna.space.
Finally I connected to the Voting app using smoothie.simplon-duna.space and it worked.

![dns_records](https://user-images.githubusercontent.com/108001918/210812132-630cc498-504f-4b29-b725-66d8f442ef4f.png)

[&#8679;](#top)

<div id='Cert-manager'/>  

### **Install Cert-manager and Jetstack (for gandi)**

[Jetstack](https://github.com/bwolf/cert-manager-webhook-gandi)

Since my ingress was working and i could connect in http to my voting app, I installed Jetstack and created a repository.

```bash
helm repo add jetstack https://charts.jetstack.io
```

Then, I installed my cert-manager to configure my certificate ([To check cert-manager last version to install](https://cert-manager.io/docs/installation/supported-releases/)).

```bash
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --set installCRDs=true --version v1.10.1 --set 'extraArgs={--dns01-recursive-nameservers=8.8.8.8:53\,1.1.1.1:53}'
```

<img width="1820" alt="Cert_manager" src="https://user-images.githubusercontent.com/108001918/210761309-393e496c-ff14-41e2-b02a-7b65fb7baeb7.png">

[&#8679;](#top)

<div id='Gsecret'/>  

### **Creation of a Gandi secret**

After cert-manager was installed, i created a secret for Gandi with the token from my gandi dns.  
*To check the token > settings from account > account and security > security.*

```bash
kubectl create secret generic gandi-credentials --from-literal=api-token='[API-TOKEN]'
```

The gandi secret was created in the default namespace because it needs to be accessed in the same namespace as the issuer (which is in the default namespace).

![gandi-secret](https://user-images.githubusercontent.com/108001918/210763575-22af59d1-812b-43fa-9017-9b9ff20b7b15.png)

[&#8679;](#top)

<div id='Webhook'/>  

### **Install cert-manager webhook for gandi**

Then i installed cert-manager-webhook for gandi so that the certificate could be linked to my DNS. I put the webhook for gandi in the cert-manager namespace because cert-manager was installed in the cert-manager namespace.

```bash
helm install cert-manager-webhook-gandi --repo https://bwolf.github.io/cert-manager-webhook-gandi --version v0.2.0 --namespace cert-manager --set features.apiPriorityAndFairness=true  --set logLevel=6 --generate-name
```

[&#8679;](#top)

<div id='Binding'/>  

### **Creation of secret role and binding for webhook**

I gave role access and created a rolebinding for the webhook (to bind gandi and the cert-manager).

```bash
kubectl create role access-secret --verb=get,list,watch,update,create --resource=secrets
```

Then i copied the webhook number from the cert-manager namespace with :

```Bash
kubectl get secrets -n cert-manager
```

![webhook](https://user-images.githubusercontent.com/108001918/210821499-2c9231b6-05a8-4a2a-964b-11c8792a9dbd.png)

And pasted it on the following command :

```bash
kubectl create rolebinding --role=access-secret default-to-secrets --serviceaccount=cert-manager:cert-manager-webhook-gandi-1672931110
```

[&#8679;](#top)

<div id='Certificate'/>  

### **Check the certificate**

Then i checked the status of my certificate with several commands :

```Bash
kubectl get certificate
```

```Bash
kubectl get orders
```

```Bash
kubectl describe orders
```

```Bash
kubectl describe challenges
```

![get_certificate](https://user-images.githubusercontent.com/108001918/210973322-b97c4836-8856-4fa1-9beb-ffb6182ff4a0.png)

[&#8679;](#top)

<div id='Scheduling'/>  

### **Scheduling**

[Scheduling a pipeline](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/scheduled-triggers?view=azure-devops&tabs=yaml)

In order to update the Voting app with its last version, I scheduled the pipeline so it would ran every hour on the hour.

![scheduling](https://user-images.githubusercontent.com/108001918/210753304-56bbd627-4139-4193-8afa-b20696251a79.png)

Then I verified that the pipeline job was a success.

![scheduling_pipeline_success](https://user-images.githubusercontent.com/108001918/210757983-4f4958a2-c718-46ec-9083-f622e9f4483f.png)

[&#8679;](#top)

<div id='Debugging'/>  

### **Pipeline debugging**

I found that I had issues with my pipeline that did not update the app version even though it was running. I added some commands to update the Kubenertes secret but the cmd line's job was failing. The commands were not read.

Then I relaunched the pipeline and got an error message saying that `the filename could not be found`. After several checks it seemed to be a credential issue. I proceeded in two steps :

- First I redid the pipeline and service connections (Kubernetes and Github) in order to get new (fresh) tokens but it did not solve the issue
- Then I added a task to get Kubernetes credentials but once again, it did not solve the issue

I started different checks and tried to point at which command, specifically, the issue happened. So I added `set -x` (that shows the process for every commands). It highlighted that there were quotation marks on the app version on the variable I had created (kubernetes secret).
I could also have used `set -e` that stops at the first error during the cmd process.

In order to remove the quotation marks from the voting app version from the VERVAR variable I created, I added  `| sed 's/"//g; s/}//g')` to the command : the quotation marks were removed from the get secret output for the variable VERCUR.

Then i added `|awk -F ':' '{print $2}'` in order to cut the output from the command and only get the second part of the secret (*{"version":"djEuMC4xMw=="}*). This gave me the following command to get the secret second part without the quotation marks and closing bracket :

``` Bash
VERCUR=$(kubectl get secret version-secret -o jsonpath='{.data}'|awk -F ':' '{print $2}' | sed 's/"//g; s/}//g')
```

Then as the secret was encoded, I used the following Bash command to decode it :

``` Bash
VERCUR=$(echo "$VAR1" | base64 -d)
```

Then I checked the result with `echo` and I had the app version display in clear without the quotation marks for the Kubenertes secret and for the curl which I then could compare. To sum up : I now have the current version, the secret version of the app and the potential difference between the two, allowing me to automatize to update (or not) the secret and the pod.

![pipeline_working_for_real](https://user-images.githubusercontent.com/108001918/216048585-9b9ae17d-b395-49e8-bd40-e65fc8c426ae.png)

_I could have also used a different way to remove the quotations marks from the variable and secret. This regex `s/^\"(.*)\"$/\1/` allows me to remove the quotation marks at the beginning and the end, but most importantly, keep 'safe' a group of characters that could contains quotation marks as well (for example a password). In this brief it is not a problem but in other situations it could become one._  
_In conclusion, as it was Alfred who created this regex to show me how he would have done it and explained it to me, I did not put it in the brief and kept my solution, but I will use it in the future. As the secret is in json has a closing bracket, I could not use this solution (but I could have use it for the curl)._

[&#8679;](#top)

<div id='Summary'/>  

### **How to get a certificate Summary**

First, apply Ingress' prerequisites and Ingress without the hosts nor TLS (DO NOT DELETE, only comment).

Then, prepare Gandi with the IP adress from Ingress. Then create the gandi secret.

Then, resettle the Host and TLS in Ingress ***DO NOT DELETE***, reapply the file. Then, install the webhook and give role access and rolebinding.

Finally, apply the Issuer yaml file and ONLY THEN, apply the certificate file (certif-space-com.yaml).

[ingress.yaml](https://github.com/simplon-lerouxDunvael/Brief_7/blob/main/Infra_K8s/ingress.yaml) -> [issuer.yaml](https://github.com/simplon-lerouxDunvael/Brief_7/blob/main/Infra_K8s/issuer.yaml) -> [certif-space-com.yaml](https://github.com/simplon-lerouxDunvael/Brief_7/blob/main/Infra_K8s/certif-space-com.yaml)

[&#8679;](#top)

<div id='ExecSummary'/>  

### **Executive summary**

[Cf. document "Executive_summary_Dun"](https://github.com/simplon-lerouxDunvael/Brief_7/blob/main/Docs/Executive_summary.docx)

[&#8679;](#top)

<div id='DAT'/>

### **Technical Architecture Document of deployed infrastructure**

[Cf. document "DAT.md"](https://github.com/simplon-lerouxDunvael/Brief_7/blob/main/Docs/DAT.md)

[&#8679;](#top)

<div id='Consumption'/>

### **Check consumption**

I tried to check the consumption for the infrastructure deployed and tests I realized, but it seems that I do not have the rights on the subscription.

```Bash
az consumption usage list --subscription a1f74e2d-ec58-4f9a-a112-088e3469febb
```

![costs](https://user-images.githubusercontent.com/108001918/211002170-0674e200-e973-4cd2-9a70-12ffa38375cd.png)

So I decided to use the Azure Calculator in order to check the consumption of this brief's resources.  

I calculated costs on several paying plans :

- [Monthly (for 12 months)](https://github.com/simplon-lerouxDunvael/Brief_7/blob/main/Docs/Costs_forecast_monthly.xlsx)
- [Yearly](https://github.com/simplon-lerouxDunvael/Brief_7/blob/main/Docs/Costs_forecast_1year.xlsx)
- [Triennially](https://github.com/simplon-lerouxDunvael/Brief_7/blob/main/Docs/Costs_forecast_3years.xlsx)

[&#8679;](#top)

<div id='UsefullCommands'/>  

### **USEFULL COMMANDS**

### **To create an alias for a command on azure CLi**

alias [WhatWeWant]="[WhatIsChanged]"  

*Example :*  

```bash
alias k="kubectl"
```

[&#8679;](#top)

### **To deploy resources with yaml file**

kubectl apply -f [name-of-the-yaml-file]

*Example :*  

```bash
kubectl apply -f azure-vote.yaml
```

[&#8679;](#top)

### **To check resources**

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

[&#8679;](#top)

### **To describe resources**

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

[&#8679;](#top)

### **To delete resources**

```bash
kubectl delete deploy --all
kubectl delete svc --all
kubectl delete pvc --all
kubectl delete pv --all
az group delete --name [resourceGroupName] --yes --no-wait
```

[&#8679;](#top)

### **To create a repository Helm and install Jetstack**

*To create the repository and install Jetstack :*

```bash
helm repo add jetstack https://charts.jetstack.io
```

*To check the repository created and Jetstack version :*

```bash
helm search repo jetstack
```

[&#8679;](#top)

### **To create a role for Gandi's secret and bind it to the webhook**

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

[&#8679;](#top)

### **To check TLS certificate in request order**

```bash
kubectl get certificate
kubectl get certificaterequest
kubectl get order
kubectl get challenge
```

[&#8679;](#top)

### **To describe TLS certificate in request order**

```bash
kubectl describe certificate
kubectl describe certificaterequest
kubectl describe order
kubectl describe challenge
```

[&#8679;](#top)

### **Get the IP address to point the DNS to nginx**

```bash
kubectl get ingress
```

[&#8679;](#top)

### **Activate the autoscaler on an existing cluster**

```bash
az aks update --resource-group b6duna --name AKSClusterd2 --enable-cluster-autoscaler --min-count 1 --max-count 8
```

[&#8679;](#top)

### **To check the auto scaling creation**

```bash
get HorizontalPodAutoscaler
```

*Example of how the results will display :*  

```bash
horizontalpodautoscaler.autoscaling/scaling-voteapp created
```

[&#8679;](#top)

### **To check Webhook configuration**

```bash
kubectl get ValidatingWebhookConfiguration -A
```

[&#8679;](#top)

### **Delete Webhook configuration for a role**

```bash
kubectl delete -A ValidatingWebhookConfiguration [rolename]  
```

*Example :*  

```bash
kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission
```

[&#8679;](#top)

</div>
