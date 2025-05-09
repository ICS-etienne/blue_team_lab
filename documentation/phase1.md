# Blue Team Lab
_Phase 1 – Fondation du Lab_  
Infrastructure Proxmox + Terraform + Ansible + Vault  
May 9, 2025

---

## Contents

1. [Phase 1 – Fondation du Blue Team Lab](#1-phase-1--fondation-du-blue-team-lab)  
   1.1. [Objectifs de la Phase 1](#11-objectifs-de-la-phase-1)  
   1.2. [Architecture Réseau Logique](#12-architecture-réseau-logique)  
2. [Installation de Proxmox VE](#2-installation-de-proxmox-ve)  
   2.1. [Téléchargement et Installation](#21-téléchargement-et-installation)  
   2.2. [Ajout du dépôt communautaire](#22-ajout-du-dépôt-communautaire)  
3. [Configuration Réseau Proxmox](#3-configuration-réseau-proxmox)  
   3.1. [Configuration des interfaces réseau](#31-configuration-des-interfaces-réseau)  
   3.2. [Attribution d’IP et Sécurisation](#32-attribution-dip-et-sécurisation)  
4. [Déploiement des VMs fondamentales](#4-déploiement-des-vms-fondamentales)  
   4.1. [Préparation : téléchargement de l’ISO Debian](#41-préparation--téléchargement-de-liso-debian)  
   4.2. [VM GitLab](#42-vm-gitlab)  
   4.3. [VM Vault](#43-vm-vault)  
   4.4. [VM DevOps](#44-vm-devops)  
   4.5. [Configuration OpenVPN sur DevOps](#45-configuration-openvpn-sur-devops)  
5. [Remarques Sécurité](#5-remarques-sécurité)

---

## 1. Phase 1 – Fondation du Blue Team Lab

### 1.1 Objectifs de la Phase 1
La Phase 1 a pour objectif de poser les bases du laboratoire **Blue Team**, en installant et configurant les outils nécessaires au fonctionnement du projet. Cela inclut la mise en place de **Proxmox VE**, la configuration réseau, ainsi que le déploiement des machines virtuelles de base pour **GitLab**, **Vault**, et **DevOps**.

### 1.2 Architecture Réseau Logique
Nous allons déployer un réseau multi-bridge dans Proxmox pour séparer les environnements. La configuration du réseau sera structurée comme suit :

- **vmbr0** : Réseau dédié aux outils (GitLab, Vault, DevOps avec OpenVPN)
- **vmbr1** : Réseau Gamenet 1 pour la simulation d’attaques/défenses (serveurs, clients, SIEM, etc.)
- **Proxmox Host** : Aucun réseau physique ne sera associé directement à l’hôte Proxmox pour des raisons de sécurité. Il sera accessible uniquement via VPN et par console.

### Exemple de configuration IP pour les VMs sur le réseau **vmbr0** (outils) :
- GitLab : 10.10.10.10
- Vault : 10.10.10.11
- DevOps (avec OpenVPN) : 10.10.10.12

Le réseau **vmbr1** est réservé pour les serveurs et outils du Gamenet.

---

## 2. Installation de Proxmox VE

### 2.1 Téléchargement et Installation
1. Récupérer l’ISO depuis le site officiel : [Proxmox Downloads](https://www.proxmox.com/downloads)
2. Graver l’ISO sur une clé USB à l’aide de **Rufus** (Windows) ou **dd** (Linux) :
   ```bash
    sudo dd if=proxmox-ve.iso of=/dev/sdX bs=4M status=progress
   ```
3. Démarrer le serveur depuis la clé USB et suivre les instructions pour installer Proxmox VE.

4. Après l’installation, configurez un nom d’hôte (ex. proxmox.lab ou proxmox.shl) et une adresse IP statique sur votre serveur Proxmox.

### 2.2 Ajout du dépôt communautaire

Si vous souhaitez avoir accès aux mises à jour gratuites de Proxmox, ajoutez le dépôt communautaire :
```bash
    sed -i 's/^deb/#deb /g' /etc/apt/sources.list.d/pve-enterprise.list
    echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-community.list
    apt update && apt full-upgrade -y
```

## 3. Configuration Réseau Proxmox

### 3.1 Configuration des interfaces réseau

Les interfaces réseau seront configurées comme suit pour supporter les différents réseaux :

- vmbr0 : réseau pour les outils GitLab, Vault, et DevOps
- vmbr1 : réseau pour le Gamenet (serveurs, clients, SIEM, etc.)
- L’hôte Proxmox ne possède aucune adresse IP physique sur ses bridges.

Voici la configuration des interfaces réseau de Proxmox pour assurer une séparation correcte des réseaux :
```bash
        iface enp32s0f0 inet manual
    auto vmbr0
    iface vmbr0 inet manual
        bridge-ports enp32s0f0
        bridge-stp off
        bridge-fd 0

    auto vmbr1
    iface vmbr1 inet manual
        bridge-ports enp32s0f1
        bridge-stp off
        bridge-fd 0
```

Notez que les interfaces physiques enp32s0f0 et enp32s0f1 seront utilisées pour chaque bridge virtuel respectivement, sans attribuer d’IP directement à ces interfaces.

### 3.2 Attribution d’IP et Sécurisation

Les interfaces virtuelles ne possèdent pas d’IP directement attribuées. Les machines virtuelles qui se connectent à ces bridges auront des adresses IP spécifiques, configurées dans leurs interfaces réseau internes.

Exemple de configuration IP pour une VM GitLab sur vmbr0 :
    Adresse IP : 10.10.10.10 (sur vmbr0)
    Passerelle et DNS : définis par OpenVPN ou configurés selon le réseau local.

## 4. Déploiement des VMs fondamentales

### 4.1 Préparation : téléchargement de l’ISO Debian

## 4.1 Préparation : téléchargement de l’ISO Debian

```bash
    cd /var/lib/vz/template/iso/
    wget https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-12.10.0-amd64-netinst.iso
```

### 4.2 VM GitLab

    ID VM = 101
    OS : Debian 12
    Ressources : 2 vCPU, 4 Go RAM, 40 Go disque
    Réseau : connecté à vmbr0
    IP statique : 10.10.10.1

### 4.3 VM Vault

    ID VM = 102
    OS : Debian 12
    Ressources : 2 vCPU, 2 Go RAM, 20 Go disque
    Réseau : connecté à vmbr0
    IP statique : 10.10.10.2

### 4.4 VM DevOps

    ID VM = 103
    OS : Debian 12
    Ressources : 2 vCPU, 4 Go RAM, 30 Go disque
    Réseau : connecté à vmbr0
    IP statique : 10.10.10.3
    Services : OpenVPN, Ansible, Terraform, Packer
