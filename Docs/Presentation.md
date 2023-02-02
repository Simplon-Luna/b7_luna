---
marp: true
theme: gaia
markdown.marp.enableHtml: true
paginate: true
---
<!-- backgroundImage: "linear-gradient(to bottom, #ffb7c5, #DCD0FF)" -->

<!--
_color: black
-->

# Présentation Brief 7
#### Mise à jour du service applicatif Voting App via Azure DevOps Pipeline

Dunvael et Luna
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;![width:700px height:350px](https://user-images.githubusercontent.com/108001918/215808280-0a44d894-1cff-436c-b4be-24ce92ae5598.png)


<!-- paginate: false -->

---

# SOMMAIRE  

01 - Projet
02 - Topologie
03 - Déploiement & maj de la Voting App via Azure DevOps Pipeline
04 - Outils et ressources
05 - Compréhension des outils et des logiciels
06 - Difficultés rencontrées
07 - Solutions trouvées
08 - DAT
09 - Executive Summary
10 - Costs forecast

<!-- paginate: true -->
<!--
_color: black
-->

---

## 1 - Projet

Utilisation de Pipelines d’Azure DevOps afin de tester les changements de version et de déployer automatiquement la mise à jour de l’application au besoin.

La mise à jour de la Voting App est planifiée chaque heure et doit être vérifiable via le pipeline.

<!--
_color: black
-->

---

## 2 - Topologie

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;![width:700px height:500px](https://user-images.githubusercontent.com/108001918/215785565-1c0a7fac-5c4d-46fb-8f0f-070392580336.png)

<!--
_color: black
-->

---

## 3 - Déploiement & maj de la Voting App via Azure DevOps Pipeline

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
![pipeline_process](https://user-images.githubusercontent.com/108001918/215771546-dd5bb6bd-c13e-41b7-992f-ea0a3c0b75d8.png)
  
<!--
_backgroundColor: black
_color: black
-->

--- 

## 3 - Déploiement & maj de la Voting App via Azure DevOps Pipeline

* Scheduling
* Kubernetes Login
* Command line :
  * Debugging (à enlever en prod)
  * Extraction n° de version sur Docker (actuelle)
  * Extraction n° de version Kubernetes (en usage)
  * Vérification différence entre versions + déclenchement maj
* Mise à jour du pod Kubernetes

<!--
_backgroundColor: black
_color: black
-->

--- 

## 4 - Outils et ressources 

| Ressources | Azure  | AKS  | Azure DevOps  | GitHub  | Docker |
|:---|:---:|:---:|:---:|:---:|:---:|
| Groupe de ressources  |  ✓ |  ✓ |  ✓ |  ✗ |  ✗ |
| Image | ✗  |  ✓ |  Ubuntu:Latest | ✗  |  ✓ |
| Kubernetes secret | ✗  | ✓  | ✓  | ✗  |  ✗ |
| Ingress |  ✓ | ✓  | ✗  | ✗  |  ✗ |
| Cert-manager |  ✗ | ✓ v1.10.1  | ✗  | ✗  | ✗  |

<!--
_color: black
-->

--- 

## 4 - Outils et ressources 

* Documentation Microsoft Azure
* Documentation Kubernetes
* Documentation Azure DevOps Pipeline
* Portail Azure pour l'interface graphique
* Cloudshell du portail Azure
* Moteur de recherche Google
* Visual Studio Code
* Github
* *L'aide précieuse de Quentin et de nos formateurs*

<!--
_color: black
-->

---

## 5 - Compréhension des outils et des logiciels

Un pipeline est un enchaînement de tâches qui permet de planifier les différentes étapes de déploiement du code dans un CI/CD.

Azure DevOps Pipelines permet :
* de tester le code et les changements apportés
* de produire un artefact pour le déploiement

=> Création des différentes tâches et de ce qu’elles doivent exécuter (commandes, vérifications...). Lancement du pipeline après l’avoir lié à Github et Kubernetes.

<!--
_color: black
-->

---

## 6 - Difficultés rencontrées

Nous avons rencontré plusieurs difficultés :

* Difficultés avec le certificat TLS et la connexion au nom de domaine en https
* Gestion du pipeline avec le script en Bash
* La liaison entre Azure DevOps et Azure
* Les droits/rôles sur l'AD de Simplon qui nous ont ralenties et fortement contraintes
* L'extraction et la transformation du secret afin qu'il puisse être utilisable en dehors de Kubernetes

<!--
_color: black
-->

---

## 7 - Solutions trouvées

Afin de palier aux difficultés, nous avons cherché des solutions et avons adopté différents comportements :

* Communications avec les autres membres de la formation
* Recherches sur les documentations et sites communautaires (messages d'erreur)
* Ecriture du code avec le debuggage en tête (ciblage et vérifications dans les processus (*`set -x`* et *`set -e`*))
* Des temps de pause (câlins et bisous au chaton par exemple ou pause chocolat) et blagues avec les collègues

<!--
_color: black
-->

---

## 8 - DAT

[Document d'Architecture Technique](https://github.com/simplon-lerouxDunvael/Brief_7/blob/main/Docs/DAT.md)

<!--
_color: black
-->

---

## 9 - Executive Summary

[Executive summary](https://github.com/simplon-lerouxDunvael/Brief_7/blob/main/Docs/Executive_summary.docx)

<!--
_color: black
-->

---

## 10 - Costs forecast

|Type | Mois | Année | Tri-annuel | Détails |
|---|---|---|---|---|
| Pay-as-You-Go | 34,04€ | 408,49€ | 1225,44€ | [Monthly Cost forecast](https://github.com/simplon-lerouxDunvael/Brief_7/blob/main/Docs/Costs_forecast_monthly.xlsx) |
| Pré-paiement annuel | 20,66€ | 247,95€ | 743,76€ | [Yearly Cost forecast](https://github.com/simplon-lerouxDunvael/Brief_7/blob/main/Docs/Costs_forecast_1year.xlsx) |
| Pré-paiement tri-annuel | 13,78€ | 165,37€ | 496,08€ | [Triennaly](https://github.com/simplon-lerouxDunvael/Brief_7/blob/main/Docs/Costs_forecast_3years.xlsx) |

<!--
_color: black
-->

---

## 10 - Costs forecast

Dépenses d’exploitation : 
- OPEX (en fonction de l’usage)
- CAPEX (gestion du pipeline - connexion – électricité)  

Il n’y a pas d’entretien (ou presque) car pas de machines physiques (hormis le point d’accès administrateur).

<!--
_color: black
-->

---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;![Merci_-_Noir](https://user-images.githubusercontent.com/108001918/196387576-cfcdcdda-7a6b-4021-ab84-3a06ebc95ab6.png)

<!--
_color: black
-->