# 🛡️ Sauvegarde et Restauration de VMs sur Proxmox VE (Disaster Recovery)

> **Projet :** Mise en œuvre d'une stratégie de continuité d'activité (PCA) et de reprise après sinistre (PRA) dans un environnement virtualisé imbriqué.

---

## 📋 Présentation

Ce projet démontre la mise en place d'une infrastructure résiliente sous **Proxmox Virtual Environment (VE)**. L'objectif est de maîtriser les deux piliers de la protection de données :

1.  **Les Snapshots (Instantanés) :** Pour le versioning rapide et les tests.
2.  **Les Backups (Sauvegardes) :** Pour la protection contre la perte totale de données.

Le projet a été réalisé dans un environnement de **Virtualisation Imbriquée (Nested Virtualization)**, simulant un Datacenter réel à partir d'un poste de travail standard.

---

## 🏗️ Architecture du Lab

L'infrastructure repose sur une architecture en couches :

*   **Hôte Physique :** PC Windows 11 (Réseau Wi-Fi/Ethernet).
*   **Hyperviseur Niveau 1 :** VMware Workstation Pro/Player (Configuration NAT/Bridge).
*   **Hyperviseur Niveau 2 :** Proxmox VE 8.x (IP Statique).
*   **VM Cible (Victime) :** Ubuntu Server 24.04 LTS.

```mermaid
graph TD;
    A[PC Physique Windows] -->|Héberge| B[VMware Workstation];
    B -->|Virtualise| C[Proxmox VE];
    C -->|Héberge| D[VM Ubuntu Server];
    C -->|Stocke Backups| E[Disque Local];


⚙️ Prérequis & Configuration
Pour permettre à Proxmox de fonctionner dans VMware, une configuration spécifique a été nécessaire :
Activation de la virtualisation imbriquée (Virtualize Intel VT-x/EPT or AMD-V/RVI).
Configuration réseau adaptée (Bridge ou NAT) pour l'accès Internet de Proxmox.
![alt text](Lien_vers_image_vmware_settings.png)

(Configuration CPU VMware)
📸 Partie 1 : Les Snapshots (Protection à court terme)
Scénario : Modification critique du système (Simulation d'erreur humaine).
Création d'un fichier critique sur la VM Ubuntu (important_data.txt).
Prise d'un Snapshot nommé "Etat-Stable".
Incident : Suppression accidentelle du fichier via la commande rm.
Résolution : Rollback (Retour arrière) via Proxmox.
Action	Résultat
Prise du Snapshot	🟢 Succès (État figé)
Suppression Fichier	🔴 Fichier perdu
Rollback	🟢 Système restauré en < 10s
![alt text](Lien_vers_image_snapshot_tree.png)

(Vue de l'arbre des snapshots dans Proxmox)
💾 Partie 2 : Les Backups (Protection à long terme)
Scénario : Crash total du serveur ou suppression de la VM (Disaster Recovery).
Configuration du stockage de sauvegarde (vzdump sur local).
Exécution d'une sauvegarde complète (Mode Snapshot, Compression ZSTD).
Incident Majeur : Suppression totale de la VM 100 (Simulant un crash disque).
Résolution : Restauration complète depuis l'archive de sauvegarde.
Comparatif technique :
Caractéristique	Snapshot 📸	Backup 💾
Stockage	Différentiel (sur le disque VM)	Archive compressée .vma.zst (Indépendant)
Indépendance	Dépend du disque original	Autonome (peut être déplacé)
Usage	Avant mise à jour / Test	Sinistre / Archivage / Ransomware
![alt text](Lien_vers_image_backup_log.png)

(Log de succès "TASK OK" lors du backup)
![alt text](Lien_vers_image_restore_menu.png)

(Interface de restauration de la VM)
🤖 Automatisation
Pour garantir la règle du RPO (Recovery Point Objective), une tâche planifiée a été créée :
Fréquence : Toutes les 30 minutes (pour le test).
Rétention : Conservation des 2 dernières copies uniquement (pour économiser l'espace).
![alt text](Lien_vers_image_schedule.png)

(Tableau de planification des backups)
🚀 Conclusion
Ce projet a permis de valider :
La faisabilité de la virtualisation imbriquée pour des labs complexes.
La fiabilité du mécanisme de snapshot de Proxmox (basé sur QCOW2/LVM).
La robustesse des sauvegardes complètes vzdump pour la reprise après sinistre.
Améliorations futures possibles :
Mise en place de Proxmox Backup Server (PBS) pour la déduplication.
Envoi des sauvegardes vers un NAS externe ou le Cloud (Règle 3-2-1).
