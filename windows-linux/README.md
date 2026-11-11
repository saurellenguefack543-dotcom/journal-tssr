# M305 Windows et Linux

Module : M305 Windows et Linux.
Statut : en cours, depuis octobre 2026.

Cette page est le plan de mes travaux pratiques. Je viendrai cocher et documenter chaque point au fur et à mesure, avec une note ou un compte rendu dans ce dossier.

This page is the plan for my labs. I will tick and document each item as I go, with a note or a report in this folder.

## Objectif du module / Aim of the module

Apprendre à administrer un poste et un petit service, sous Windows Server et sous Linux : créer des comptes, régler les droits, maintenir le système à jour, et lire les journaux quand quelque chose cloche. L'idée, pour moi, est d'avoir les mêmes réflexes sur les deux familles de systèmes.

Learn to administer a workstation and a small service, on Windows Server and on Linux: create accounts, set permissions, keep the system up to date, and read the logs when something goes wrong. For me, the point is to build the same habits on both families of systems.

## Plan des travaux pratiques / Lab plan

### Comptes / Accounts

- [ ] Créer, modifier et désactiver un compte local sous Linux
- [ ] Créer, modifier et désactiver un compte local sous Windows
- [ ] Comprendre la différence entre un compte local et un compte de domaine (Active Directory)
- [ ] Assigner un utilisateur à un groupe, et vérifier l'effet sur l'ouverture de session
- [ ] Create, modify and disable a local account on Linux
- [ ] Create, modify and disable a local account on Windows
- [ ] Understand the difference between a local account and a domain account (Active Directory)
- [ ] Assign a user to a group, and check the effect on sign-in

### Droits / Permissions

- [ ] Lire et modifier les droits sur un fichier et un répertoire sous Linux (`ls -l`, `chmod`, `chown`, groupes)
- [ ] Lire et modifier les droits NTFS sous Windows, et distinguer un droit partagé d'un droit NTFS
- [ ] Vérifier qu'un compte sans privilège ne peut pas faire une action d'administration
- [ ] Read and change permissions on a file and a directory on Linux (`ls -l`, `chmod`, `chown`, groups)
- [ ] Read and change NTFS permissions on Windows, and tell a share permission from an NTFS permission
- [ ] Check that an unprivileged account cannot perform an administrative action

### Mises à jour / Updates

- [ ] Appliquer les mises à jour d'un poste Ubuntu avec `apt` et vérifier ce qui a été installé
- [ ] Appliquer les mises à jour d'un poste ou d'un serveur Windows et lire l'historique
- [ ] Noter comment revenir en arrière si une mise à jour casse un service
- [ ] Apply updates on an Ubuntu machine with `apt` and check what was installed
- [ ] Apply updates on a Windows workstation or server and read the history
- [ ] Note how to roll back if an update breaks a service

### Journaux / Logs

- [ ] Lire les journaux Linux (`journalctl`, fichiers de `/var/log`) pour retrouver une ouverture de session et une erreur de service
- [ ] Lire l'Observateur d'événements Windows (journaux Sécurité, Système, Application)
- [ ] Relier une erreur visible par l'utilisateur à une ligne de journal
- [ ] Read Linux logs (`journalctl`, files in `/var/log`) to find a sign-in and a service error
- [ ] Read the Windows Event Viewer (Security, System, Application logs)
- [ ] Tie an error the user can see to a log line

## Prochaine note / Next note

La première note de ce dossier suivra le prochain TP du module. En attendant, le compte rendu d'un dépannage Linux déjà réalisé se trouve ici : [Installation de Packet Tracer sur Ubuntu et récupération de compte via GRUB](../linux/01-packet-tracer-ubuntu.md).

The first note in this folder will follow the next lab in the module. In the meantime, a Linux troubleshooting report already written is here: [Installing Packet Tracer on Ubuntu and recovering an account through GRUB](../linux/01-packet-tracer-ubuntu.md).
