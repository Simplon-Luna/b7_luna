# Technical Architectural Document
## Functions requirements
Users needs to be able to vote and reset votes on the Voting App application and Voting App needs to be highly available.

## Non-functionnal Needs
| Element | Value | Comments |
| ------- | ------ | ------------ |
| Provider | Azure | - Value and cost-effectiveness <br> - Compatibility <br> - Security |
| Service | AKS | - Auto upgrades <br> - Patching <br> - Self-healing |
| Gateway | Ingress | Integrated in AKS |
| BDD | Redis | - Good performance, low latency <br> - Flexible data structures <br> - Open Source |
| Application | VotingApp | Required and needs to be able to scale horizontally and have a single domain name |

## Function Schematic
![ ](https://github.com/Simplon-Luna/b6_Luna/blob/main/Pics/DAT%20Diagram%201.png)

### Environment list
| Nom | Type | Caractéristiques |
| --- | ---- | ---------------- |
| Azure | Provider | Public Cloud |
| AKS | Kubernetes Cluster | Cluster K8s Azure |

## Applicative Schematic
![ ](https://github.com/Simplon-Luna/b6_Luna/blob/main/Pics/DAT%20Diagram%202.png)

### Network Flux Matrix
| Source | Destination | Protocol | Port |
|------------- | ----------- | -------- | ---- |
| Ingress Gateway | Service votingApp | TCP | 80 |
| Service votingApp | Pods votingApp | TCP | 80 |
| Pods votingApp | Service votingApp | TCP | 80 |
| Service votingApp | Service Redis | TCP | 6379 |
| Service Redis  | Pod Redis | TCP | 6379 |
## Architecture Schematic
![ ](https://github.com/Simplon-Luna/b6_Luna/blob/main/Pics/Cloud%20Architecture%202.png)
## Architecture Decisions
| Topic | Content |
| ----------------------- | ------------------------ |
| Architecture Decision | Use of a Gateway |
| Problem | What Gateway to use? |
| Hypothesis | Use of Application Gateway |
| Alternatives | A: Use of Azure Application Gateway <br> B: Use of Ingress in Kubernetes |
| Decision | Option B is selected |
| Reasoning | 1- Easier potential transition to another public cloud provider <br> 2- Less dependant on Azure specific infrastructure and services |
| Implication | Generation and incorporation of the TLS certificate within Kubernetes |