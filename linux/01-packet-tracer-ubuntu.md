# Installation de Packet Tracer sur Ubuntu et récupération de compte via GRUB

# Installing Packet Tracer on Ubuntu and recovering an account through GRUB

Date : automne 2026.
Contexte : mon poste de travail, un Dell Latitude E5250 sous Ubuntu, pour les travaux pratiques de réseaux (Cisco Packet Tracer 9.0.1) et l'administration Linux.

Date: autumn 2026.
Context: my workstation, a Dell Latitude E5250 running Ubuntu, used for networking labs (Cisco Packet Tracer 9.0.1) and Linux administration.

Ce compte rendu décrit deux incidents que j'ai réellement rencontrés et résolus sur cette machine. J'écris la démarche telle que je l'ai suivie, pour pouvoir la refaire et pour montrer comment je diagnostique un blocage.

This report describes two incidents I actually ran into and solved on this machine. I write the steps I followed, so that I can repeat them and so that it is clear how I diagnose a blockage.

---

## Incident 1 : Packet Tracer refuse de démarrer / Packet Tracer will not start

### Symptôme / Symptom

Après l'installation de Cisco Packet Tracer 9.0.1 sur Ubuntu, le lancement échouait tout de suite. Le premier message était :

After installing Cisco Packet Tracer 9.0.1 on Ubuntu, the application failed immediately on launch. The first message was:

```text
libOpenGL.so.0: cannot open shared object file: No such file or directory
```

Une fois cette bibliothèque en place, un second message est apparu :

Once that library was in place, a second message appeared:

```text
no Qt platform plugin could be initialized (xcb)
```

Packet Tracer ne s'ouvrait toujours pas. L'interface graphique ne s'affichait pas du tout.

Packet Tracer still would not open. The graphical interface did not appear at all.

### Diagnostic / Diagnosis

Le premier message est assez direct : au démarrage, le binaire cherche la bibliothèque partagée `libOpenGL.so.0` et ne la trouve pas dans les chemins habituels. Ce n'est pas une erreur interne à Packet Tracer. C'est une dépendance manquante sur le système. Sur Ubuntu, ce fichier est fourni par le paquet `libopengl0`, souvent accompagné des paquets Mesa qui gèrent le rendu OpenGL.

The first message is fairly direct: at startup, the binary looks for the shared library `libOpenGL.so.0` and does not find it in the usual paths. This is not an internal Packet Tracer fault. It is a missing system dependency. On Ubuntu, that file is provided by the `libopengl0` package, often together with the Mesa packages that handle OpenGL rendering.

Le second message vient de Qt, la bibliothèque graphique utilisée par Packet Tracer. « xcb » est le greffon qui relie Qt au serveur d'affichage X11. S'il ne peut pas s'initialiser, c'est en général qu'il manque des bibliothèques XCB, typiquement `libxcb-xinerama0` et des paquets proches. En relisant les journaux et en cherchant les fichiers manquants, j'ai confirmé que mon installation Ubuntu n'avait pas ces dépendances, alors que Packet Tracer les attend.

The second message comes from Qt, the graphical toolkit Packet Tracer uses. "xcb" is the plugin that connects Qt to the X11 display server. If it cannot initialise, some XCB libraries are usually missing, typically `libxcb-xinerama0` and related packages. After reading the logs and looking for the missing files, I confirmed that my Ubuntu install did not have these dependencies, even though Packet Tracer expects them.

Je n'ai pas réinstallé Packet Tracer tout de suite. Les messages d'erreur pointaient déjà vers des bibliothèques absentes, pas vers un paquet Cisco corrompu.

I did not reinstall Packet Tracer straight away. The error messages already pointed to missing libraries, not to a corrupted Cisco package.

### Correction / Fix

J'ai installé les bibliothèques manquantes avec `apt` :

I installed the missing libraries with `apt`:

```bash
sudo apt update
sudo apt install libopengl0 libgl1-mesa-dri libglx-mesa0 libglu1-mesa
sudo apt install libxcb-xinerama0 libxcb-cursor0 libxkbcommon-x11-0
```

Après l'installation de `libopengl0` et des paquets Mesa, l'erreur `libOpenGL.so.0` a disparu. Après l'installation de `libxcb-xinerama0` et des bibliothèques XCB nécessaires au greffon Qt, Packet Tracer 9.0.1 s'est lancé normalement sur le Dell Latitude E5250.

After installing `libopengl0` and the Mesa packages, the `libOpenGL.so.0` error was gone. After installing `libxcb-xinerama0` and the XCB libraries required by the Qt plugin, Packet Tracer 9.0.1 started normally on the Dell Latitude E5250.

### Leçon / Lesson

Quand un programme graphique refuse de démarrer, je lis d'abord le message exact. « cannot open shared object file » veut dire qu'une bibliothèque système manque ; la réponse est d'identifier le paquet qui la fournit, pas de réinstaller l'application au hasard. Packet Tracer suppose un ensemble de dépendances que Ubuntu n'installe pas toujours par défaut, surtout sur une machine un peu ancienne. Je garde cette liste sous la main pour un prochain poste.

When a graphical program refuses to start, I read the exact message first. "cannot open shared object file" means a system library is missing; the answer is to identify the package that provides it, not to reinstall the application at random. Packet Tracer assumes a set of dependencies that Ubuntu does not always install by default, especially on an older machine. I am keeping this list for the next workstation.

---

## Incident 2 : perte d'accès au compte Linux / Loss of access to the Linux account

### Symptôme / Symptom

Je n'arrivais plus à ouvrir une session avec mon compte utilisateur sur Ubuntu : le mot de passe n'était plus accepté. La machine démarrait normalement jusqu'à l'écran de connexion, mais je restais bloquée à cette étape. Je n'avais plus d'accès graphique, et donc plus d'accès à mes travaux.

I could no longer sign in to my Ubuntu user account: the password was no longer accepted. The machine booted normally as far as the login screen, but I was stuck there. I no longer had a graphical session, and therefore no access to my work.

### Diagnostic / Diagnosis

Le système, lui, démarrait. Ce n'était donc pas un disque inaccessible ni un noyau qui refusait de charger. Le problème se situait au niveau du compte : mot de passe oublié, modifié par erreur, ou base des comptes incohérente. Sur Linux, on peut réinitialiser le mot de passe d'un utilisateur depuis un interpréteur root, à condition de pouvoir entrer en mode de récupération. GRUB, le chargeur de démarrage, propose cette entrée.

The system itself was booting. So this was not an unreachable disk, nor a kernel that failed to load. The problem was at the account level: a forgotten password, one changed by mistake, or an inconsistent account database. On Linux, a user's password can be reset from a root shell, provided one can enter recovery mode. GRUB, the boot loader, offers that entry.

### Correction / Fix

J'ai redémarré le poste et, à l'écran GRUB, j'ai choisi les options avancées, puis le mode de récupération (recovery mode). Dans le menu de récupération, j'ai ouvert un interpréteur de commandes root.

I rebooted the machine and, at the GRUB screen, selected Advanced options, then recovery mode. From the recovery menu, I opened a root shell.

Le système de fichiers racine est souvent monté en lecture seule dans ce mode, pour éviter d'écrire par accident. Avant de changer quoi que ce soit, je l'ai remonté en lecture-écriture :

The root file system is often mounted read-only in this mode, to avoid accidental writes. Before changing anything, I remounted it read-write:

```bash
mount -o remount,rw /
```

J'ai ensuite réinitialisé le mot de passe de mon compte, puis redémarré :

I then reset my account password and rebooted:

```bash
passwd mon_utilisateur
reboot
```

Après le redémarrage, j'ai pu ouvrir une session normalement avec le nouveau mot de passe.

After the reboot, I was able to sign in normally with the new password.

### Leçon / Lesson

Perdre l'accès à son propre compte n'oblige pas à réinstaller le système. Le mode de récupération GRUB donne un interpréteur root précisément pour ce genre de situation. Deux précautions comptent : remonter la partition racine en lecture-écriture avant `passwd`, et ne jamais laisser un poste sans autre moyen d'administration. Sur un poste professionnel, cette procédure doit rester contrôlée : elle montre aussi pourquoi l'accès physique à une machine est déjà un accès à protéger.

Losing access to my own account does not mean I have to reinstall the system. GRUB recovery mode provides a root shell for exactly this kind of situation. Two precautions matter: remounting the root partition read-write before `passwd`, and never leaving a machine with no other administrative path. On a work workstation, this procedure must stay under control: it also shows why physical access to a machine is already an access path that needs protecting.
