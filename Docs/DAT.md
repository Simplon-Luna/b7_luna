# Technical Architectural Document
## Functions requirements
Les utilisateurs doivent pouvoir voter sur l’application Voting App (hautement disponible, sécurisée et accessible via https et nom de domaine) qui doit être mise à jour automatiquement toutes les heures.

## Non-functionnal Needs
| Element | Value | Comments |
| ------- | ------ | ------------ |
| Provider | Azure | - Value and cost-effectiveness <br> - Compatibility <br> - Security |
| Service | AKS | - Auto upgrades <br> - Patching <br> - Self-healing |
| Gateway | Ingress | Integrated in AKS |
| BDD | Redis | - Good performance, low latency <br> - Flexible data structures <br> - Open Source |
| Application | VotingApp | Required and needs to be able to scale horizontally and have a single domain name |
| Azure DevOps Pipeline | Deployment | To check and update the Voting App version |

## Function Schematic

<img src="https://user-images.githubusercontent.com/108001918/215785382-d165ece4-818c-4f9b-818c-12ff59d6c6eb.png" width="350">

### Environment list
| Nom | Type | Caractéristiques |
| --- | ---- | ---------------- |
| Azure | Provider | Public Cloud |
| AKS | Kubernetes Cluster | Cluster K8s Azure |
| Azure DevOps | Pipeline Manager | CI/CD |

## Applicative Schematic

![application_function](https://user-images.githubusercontent.com/108001918/215785565-1c0a7fac-5c4d-46fb-8f0f-070392580336.png)

### Network Flux Matrix
| Source | Destination | Protocol | Port |
|------------- | ----------- | -------- | ---- |
| Ingress Gateway | Service votingApp | TCP | 80 |
| Service votingApp | Pods votingApp | TCP | 80 |
| Pods votingApp | Service votingApp | TCP | 80 |
| Service votingApp | Service Redis | TCP | 6379 |
| Service Redis  | Pod Redis | TCP | 6379 |
| Azure DevOps Pipeline | Github connection services | ✗  | ✗  |
| Azure DevOps Pipeline | Kubenetes connection services | ✗  | ✗  |

## Architecture Schematic

![pipeline_process](https://user-images.githubusercontent.com/108001918/215771546-dd5bb6bd-c13e-41b7-992f-ea0a3c0b75d8.png)

## Architecture Decisions
| Topic | Content |
| ----------------------- | ------------------------ |
| Architecture Decision | Use of a pipeline |
| Problem | Which pipeline Manager to use? |
| Hypothesis | Use of Azure qualified pipeline |
| Alternatives | A: Use of Gitlab CI Actions <br> B: Use of Jenkins <br> C: Use of Azure Devops |
| Decision | Option C is selected |
| Reasoning | 1- Better integration with Azure Cloud Services (AKS) <br> 2- No need of external VMs for Pipeline Management |
| Implication | Incorporation of code in Azure DevOps Pipeline and deployment of infrastructure |