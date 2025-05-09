# Blue Team Lab
Projet HIP x SHL de conception d'un exercice technique de gestion de crise cyber sur un déploiement d'infrastructure automatisé.

<p align="center">
  <img src="/media/hip.png" alt="Logo HIP" height="100" style="margin-right: 20px;"/>
  <img src="/media/shl.png" alt="Logo SHL" height="100"/>
</p>

**Conception Générale**  
Infrastructure Proxmox + Terraform + Ansible + packer + GitLab + Vault + GameNet de l'exercie

---

## Sommaire

1. [Vision Globale](#1-vision-globale)  
2. [Composants Clés et Outils Recommandés](#2-composants-clés-et-outils-recommandés)  
3. [Vision Réseau et Sécurité](#3-vision-réseau-et-sécurité)  
4. [Vision Gamenet](#4-vision-gamenet)  
5. [Déroulé Logique du Projet](#5-déroulé-logique-du-projet)  
6. [Accès pour Développeurs et Joueurs](#6-accès-pour-développeurs-et-joueurs)  
7. [Base Documentaire à Produire](#7-base-documentaire-à-produire)  
8. [Repos Git – Structure type](#8-repos-git--structure-type)  

---

## 1. Vision Globale

Le projet **Blue Team Lab** vise à déployer une infrastructure d’entraînement cyber défensif automatisée, réplicable et isolée, à l’aide de l’hyperviseur **Proxmox VE** et d’outils **IaC** (Infrastructure as Code).

**Objectifs :**

- Automatiser la création de réseaux isolés simulant une entreprise vulnérable  
- Gérer la configuration complète avec GitLab, Ansible, Terraform, Vault  
- Permettre à des développeurs et participants d’accéder de manière sécurisée  

---

## 2. Composants Clés et Outils Recommandés

- **Virtualisation** : Proxmox VE  
- **Provisioning** : Terraform, Packer, Ansible  
- **CI/CD** : GitLab (plan gratuit), runner local  
- **Gestion des secrets** : HashiCorp Vault (sur VM dédiée)  
- **Simulation de vie** : GHOSTS (CMU-SEI)  
- **Simulation d’attaques** : Atomic Red Team, Caldera  
- **VPN** : OpenVPN  
- **2FA** : FreeOTP (accès Proxmox Web UI)  
- **SIEM / Logs** : ELK Stack, Splunk Free  

---

## 3. Vision Réseau et Sécurité

### Interfaces réseau de l’hôte Proxmox

| Interface     | Bridge   | Rôle                                 |
|---------------|----------|--------------------------------------|
| enp32s0f0     | vmbr0    | Réseau proxmox + DevOps (10.10.10.0/24)  
| enp32s0f1     | vmbr1    | Réseau Gamenet (192.168.11.0/24 + 192.168.12.0/24)  
| *(optionnel)* | vmbr2,3, ...n  | Réseaux Gamenet supplémentaires si besoin  (192.168.n1.0/24 + 192.168.n2.0/24)

### Sécurité réseau

- **Firewall Proxmox**
- **1 VM Bastion par Gamenet**, accès uniquement via VPN
- **Pare-feu local activé sur chaque bastion**
- **UI Web Proxmox uniquement accessible via VPN et 2FA (FreeOTP)**
- **Pas d’accès direct au réseau DevOps depuis les VMs Gamenet**

---

## 4. Vision Gamenet

Chaque **Gamenet** simule un environnement d’entreprise interne :

- 1x Windows Server (AD, DNS, DHCP, GPO)  
- 3x Clients Windows 11 avec agents GHOSTS  
- 1x Attaquant (Debian)  
- 5x Serveurs Linux (SIEM, logs : ELK, Splunk, syslog, etc.)  

**Accès Joueur** : VPN → Bastion → Rebond SSH → Accès aux VMs.  
*Aucun accès direct à Proxmox ou aux réseaux de gestion.*

---

## 5. Déroulé Logique du Projet

### Phase 1 – Fondation

- Installer Proxmox et configurer `bridge` et `NAT` pour DevOps et Gamenet.
- Déployer les VMs GitLab, Vault, Ansible, Packer, Terraform.
- Configurer OpenVPN pour les accès sécurisés des développeurs.

### Phase 2 – Infrastructure as Code (IaC)

- Créer les templates Packer (Windows 11, Debian, Windows Server)
- Déployer la topologie réseau du gamnet via Terraform
- Configurer les VMs via Ansible

### Phase 3 – Sécurité

- UI Proxmox accessible uniquement via VPN + 2FA
- Secrets gérés par Vault : SSH keys, tokens Terraform, mots de passe
- GitLab CI/CD déclenche les déploiements

### Phase 4 – Exercice Cyber

- Déploiement de scénarios d’attaques (Atomic Red Team, Caldera)
- Accès joueur via VPN + bastion
- Analyse des logs, défense, patchs en temps réel

---

## 6. Accès pour Développeurs et Joueurs

### Développeurs (remote)

- Accès VPN (OpenVPN) → `vmbr0` (réseau 10.10.10.0/24)  
- UI Web Proxmox via FreeOTP (2FA)  
- Déploiement GitLab CI + runners locaux  
- Accès SSH et tokens temporaires via Vault  
- Aucun rebond possible vers VMs Gamenet

### Joueurs

- VPN → Bastion → SSH vers VMs Gamenet  
- Aucun accès aux VMs de DevOps ou à Proxmox  
- Accès restreint au réseau `192.168.1.0/24` via jump host  

---

## 7. Base Documentaire à Produire

1. Schéma réseau par Gamenet  
2. Manuel d’utilisation DevOps  
3. Fiches scénarios d’attaques cyber  
4. Politique de sécurité et conformité  
5. Guide utilisateur Vault  
6. Glossaire des VMs  
7. Procédures de reset / redeploiement d’un Gamenet  
8. Plaquette de présentation de l’exercice  

---

## 8. Repos Git – Structure type

<pre Blue Team Lab
├── packer/ 
│ ├── windows11.json 
│ └── debian12.json 
├── terraform/ 
│ ├── main.tf 
│ └── modules/ 
├── ansible/ 
│ ├── playbooks/ 
│ ├── roles/ 
│ └── inventories/ 
├── vault/ 
│ ├── policies/ 
│ └── init.sh 
├── gitlab-ci/ 
│ └── .gitlab-ci.yml 
└── docs/ 
 </pre>

