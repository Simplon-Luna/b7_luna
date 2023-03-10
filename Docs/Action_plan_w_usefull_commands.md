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
###### [17 - Scheduling](#Scheduling)
###### [18 - Add Ingress](#Ingress)
###### [19 - Install Cert-manager and Jetstack (for gandi)](#Cert-manager)
###### [20 - Creation of a Gandi secret](#Gsecret)
###### [21 - Install cert-manager webhook for gandi](#Webhook)
###### [22 - Creation of secret role and binding for webhook](#Binding)
###### [23 - Check the certificate](#Certificate)
###### [24 - How to get a certificate Summary](#Summary)
###### [25 - Executive summary](#ExecSummary)
###### [26 - Technical Architecture Document of deployed infrastructure](#DAT)
###### [27 - Check consumption](#Consumption)

<div id='Scrum'/>  

### 0. Scrum quotidien

Scrum Master = Me, myself and I
Daily personnal reactions with reports and designations of first tasks for the day.

Frequent meeting with other coworkers to study solutions to encountered problems together.

[scrums](https://github.com/simplon-lerouxDunvael/Brief_7/blob/main/Plans_et_demarches/Scrum.md)

[&#8679;](#top)

<div id='Docs'/>  

#### **Kubernetes, AKS and Azure Pipelines doc reading**
Lecture des documentations afin de d??terminer les fonctionnements, pr??requis et outils/logiciels n??cessaires pour remplir les diff??rentes t??ches du Brief 7.

[&#8679;](#top)  

<div id='Topology'/>  

#### **Architecture Topology**
Infrastructure Plannifi??e

*Sch??ma r??alis?? dans le cas plus g??n??ral o?? les pods ne sont pas dans le m??me node.*
*Les pods sont sch??matis??s par un seul objet m??me s'ils peuvent repr??senter plusieurs r??plicas.*

```mermaid
flowchart TD

user(Utilisateurs)
web{Internet}
    subgraph AZ [AZURE]
        
        subgraph Cluster [Cluster Kubernetes]
            cluster{{Cluster IP}}
            Nginx{{Ingress}}

            subgraph n1 [Node 1]
                KT-temp1((Redis))
            end

            subgraph n2 [Node 2]
                KT-temp2((Voting App))
            end
        end

        subgraph BDD [Stockage Redis]
            Stock[(Persistent Volume Claim)]
        end

    end


KT-temp1 -.-> Stock
user -.-> |web| web -.-> |web| Nginx
KT-temp1 <--> cluster <--> KT-temp2
Nginx <--> cluster

    classDef rouge fill:#faa,stroke:#f66,stroke-width:3px,color:#002;
    class Cluster, rouge;
    classDef bleu fill:#adf,stroke:#025,stroke-width:4px,color:#002;
    class AZ, bleu;
    classDef fuschia fill:#f0f,stroke:#c0d,stroke-width:2px,color:#003;
    class BDD, fuschia;
    classDef vert fill:#ad5,stroke:#ad5,stroke-width:2px,color:#003;
    class cluster,Nginx, vert;
    classDef rose fill:#faf,stroke:#faf,stroke-width:2px,color:#003;
    class Stock, rose;
    classDef jaune fill:#fe5,stroke:#fe5,stroke-width:2px,color:#003,stroke-dasharray: 5 5;
    class user, jaune;
    classDef blanc fill:#eff,stroke:#eff,stroke-width:2px,color:#003;
    class web, blanc;
    classDef gris fill:#bab,stroke:#bab,stroke-width:2px,color:#003;
    class n1,n2, gris;
    classDef bleugris fill:#bbe,stroke:#bbe,stroke-width:2px,color:#003;
    class KT-temp1,KT-temp2, bleugris;

```

[&#8679;](#top)  

<div id='Ressources'/>  

#### **Resource List**

-----------
| Ressources | Cluster AKS | Redis |  Voting App |
| :--------: | :--------: | :--------: | :--------: |
| Azure service | ??? | ??? | ??? |
| Azure DevOps | ??? | ??? | ??? |
| ressource groupe | ??? |??? | ??? |
| SSH (port) | N/A | 6379 | 80 |
| CPU | N/A | 100m-250m | 100m-250m |
| M??moire | N/A | 128mi-256mi | 128mi-256mi |
| Image | N/A | redis:latest  | simplonasa/azure_voting_app:v1.0.11 |
| Load Balancer | N/A | ??? puis ??? | ??? |
| ClusterIP | N/A | ??? puis ??? | ??? |
| Kebernetes secret | ??? | ??? | ??? |
| Storage secret | ??? | ??? | ??? |
| Storage account (Standard LRS) | N/A | ??? | ??? |
| Persistent Volume | N/A | ??? | ??? |
| Persistent Vol. Claim (3Gi)| N/A | ??? | ??? |
| Ingress | ??? | ??? | ??? |
| Nginx | ??? | ??? | ??? |
| DNS | ??? | N/A | ??? |
| Cert-manager | N/A | N/A | v1.10.1 |
| Certificat TLS | N/A | N/A | ??? |


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

<div id='Scheduling'/>  

### **Scheduling**

[Scheduling a pipeline](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/scheduled-triggers?view=azure-devops&tabs=yaml)

In order to update the Voting app with its last version, I scheduled the pipeline so it would ran every hour on the hour.

![scheduling](https://user-images.githubusercontent.com/108001918/210753304-56bbd627-4139-4193-8afa-b20696251a79.png)

Then I verified that the pipeline job was a success.

![scheduling_pipeline_success](https://user-images.githubusercontent.com/108001918/210757983-4f4958a2-c718-46ec-9083-f622e9f4483f.png)

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

<div id='Summary'/>  

### **How to get a certificate Summary**

First, apply Ingress' prerequisites and Ingress without the hosts nor TLS (DO NOT DELETE, only comment).

Then, prepare Gandi with the IP adress from Ingress. Then create the gandi secret.

Then, resettle the Host and TLS in Ingress ***DO NOT DELETE***, reapply the file. Then, install the webhook and give role access and rolebinding.

Finally, apply the Issuer yaml file and ONLY THEN, apply the certificate file (certif-space-com.yaml).

[ingress.yaml](https://github.com/simplon-lerouxDunvael/Brief_7/blob/main/Infra_K8s/ingress.yaml) -> [issuer.yaml](https://github.com/simplon-lerouxDunvael/Brief_7/blob/main/Infra_K8s/issuer.yaml) -> [certif-space-com.yaml](https://github.com/simplon-lerouxDunvael/Brief_7/blob/main/Infra_K8s/certif-space-com.yaml)

[&#8679;](#top)

<div id='UsefullCommands'/>  

### **USEFULL COMMANDS**

### **To create an alias for a command on azure CLi**

alias [WhatWeWant]="[WhatIsChanged]"  

_Example :_  

```bash
alias k="kubectl"
```

[&#8679;](#top)

### **To deploy resources with yaml file**

kubectl apply -f [name-of-the-yaml-file]

_Example :_  

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

_To keep verifying the resources add --watch at the end of the command :_

_Example :_

```bash
kubectl get services --watch
```

_To check the resources according to their namespace, add --namespace after the command and the namespace's name :_

_Example :_

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

_To specify which resource needs to be described just put the resource ID at the end of the command._

_Example :_

```bash
kubectl describe svc redis-service
```

_To access to all the logs from all containers :_

```bash
kubectl logs podname --all-containers
```

_To access to the logs from a specific container :_

```bash
kubectl logs podname -c [container's-name]
```

_To list all events from a specific pod :_

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

_To create the repository and install Jetstack :_

```bash
helm repo add jetstack https://charts.jetstack.io
```

_To check the repository created and Jetstack version :_

```bash
helm search repo jetstack
```

[&#8679;](#top)

### **To create a role for Gandi's secret and bind it to the webhook**

_To create the role :_

```bash
kubectl create role [role-name] --verb=[Authorised-actions] --resource=[Authorised-resource]
```

_Example :_  

```bash
kubectl create role access-secrets --verb=get,list,watch,update,create --resource=secrets
```

_To bind it :_  

```bash
kubectl create rolebinding --role=[role-name] [role-name] --serviceaccount=[group]:[group-item]
```

_Example :_  

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

_Example of how the results will display :_  

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

_Example :_  

```bash
kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission
```

[&#8679;](#top)

<div id='ExecSummary'/>  

### **Executive summary**

[Cf. document "Executive_summary_Dun"]()

[&#8679;](#top)

<div id='DAT'/> 

### **Technical Architecture Document of deployed infrastructure**

[Cf. document "DAT.md"](https://github.com/simplon-lerouxDunvael/Brief_7/blob/main/Docs/DAT.md)

[&#8679;](#top)

<div id='Consumption'/> 

### **Check consumption**

I tried to check the consumption for the infrastructure deployed and tests I realized, but it seems that I do not have the rights on the subscription. Therefore, I cannot provide the costs for this Brief.

```Bash
az consumption usage list --subscription a1f74e2d-ec58-4f9a-a112-088e3469febb
```

![costs](https://user-images.githubusercontent.com/108001918/211002170-0674e200-e973-4cd2-9a70-12ffa38375cd.png)

[&#8679;](#top)

</div>