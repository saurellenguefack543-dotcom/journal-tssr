---
layout: post
title: "Packet Tracer sur Ubuntu : installation, erreurs et récupération de session"
date: 2026-11-09 18:00:00 +0100
tags:
  - Linux
excerpt: "Sur mon Dell Latitude, Packet Tracer refusait de s'ouvrir, puis je n'arrivais plus à ouvrir une session Ubuntu. Voici le récit, avec les captures de mon écran, et ce que j'en retiens."
---

J'utilise un Dell Latitude E5250 sous Ubuntu pour mes travaux pratiques de réseaux. Le 5 novembre, j'y installe Cisco Packet Tracer 9.0.1. Sur le papier, c'est un paquet `.deb` à installer, puis un logiciel à ouvrir. Sur cette machine, ça ne l'a pas été : d'abord des bibliothèques manquantes, ensuite une perte d'accès à mon propre compte. Voici la chronologie réelle, avec les photos de mon écran.

<!--more-->

## Le paquet .deb et la licence Cisco

Dans mon dossier Téléchargements, j'ai lancé l'installation du paquet officiel :

```bash
sudo apt install ./CiscoPacketTracer_901_Ubuntu_64bit.deb
```

Le programme d'installation s'est arrêté sur le contrat de licence. Le texte s'affiche dans un pager, ce petit lecteur de terminal où l'on fait défiler une page. En bas de l'écran : `Press q to quit`.

<figure>
  <img src="{{ '/assets/images/linux-packet-tracer/00a-eula-texte.jpg' | relative_url }}" alt="Écran de mon Dell Latitude : le contrat de licence de Packet Tracer s'affiche dans le terminal, avec la mention Press q to quit.">
  <figcaption>La licence Cisco s'affiche dans un pager. Pour quitter cette vue et continuer, il faut appuyer sur q.</figcaption>
</figure>

Après le pager, un menu me demande de choisir : `1` pour relire le texte, `2` pour accepter, `3` pour refuser. La barre de progression est alors à 90 %.

<figure>
  <img src="{{ '/assets/images/linux-packet-tracer/00b-eula-choix.jpg' | relative_url }}" alt="Terminal Ubuntu : menu de la licence Cisco, options 1, 2 ou 3, barre de progression à 90 %.">
  <figcaption>Le menu de la licence, pendant que l'installation est déjà à 90 %. J'ai ensuite choisi 2, pour accepter.</figcaption>
</figure>

J'ai saisi `2`. Le terminal a confirmé `Accept EULA (selected)` et l'installation a pu se terminer.

<figure>
  <img src="{{ '/assets/images/linux-packet-tracer/00c-installation-deb-90.jpg' | relative_url }}" alt="Terminal Ubuntu : licence acceptée, option 2 sélectionnée, installation du paquet .deb encore à 90 %.">
  <figcaption>Licence acceptée. Le paquet continue de s'installer.</figcaption>
</figure>

## Un avertissement _apt, puis libOpenGL.so.0 introuvable

Une fois le paquet en place, le terminal a affiché plusieurs messages. D'abord des avertissements `xdg-icon-resource` : le système n'a pas trouvé de dossier d'icônes accessible en écriture. Puis cette notice, plus inquiétante au premier regard :

```text
Notice: Download is performed unsandboxed as root as file
'/home/saun/Downloads/CiscoPacketTracer_901_Ubuntu_64bit.deb'
couldn't be accessed by user '_apt' - pkgAcquire::Run (13: Permission denied)
Warning: Operation was interrupted before it could finish
```

Ce n'est pas forcément une installation ratée. Pour télécharger ou lire un paquet, `apt` utilise un utilisateur dédié, `_apt`, afin de ne pas tout faire en root. Quand le fichier `.deb` se trouve dans mon dossier personnel, `_apt` n'a souvent pas le droit de le lire. `apt` se rabat alors sur une lecture en root, hors de son isolement habituel, d'où le message « unsandboxed ». Pour éviter cet avertissement, on peut copier le paquet dans `/tmp` avant de l'installer, un emplacement que `_apt` peut lire. La ligne « Operation was interrupted before it could finish » m'a aussi fait hésiter. Dans mon cas, le programme était bien présent : le vrai blocage est venu juste après, au lancement.

J'ai lancé Packet Tracer. Le logiciel n'a ouvert aucune fenêtre. Juste une ligne :

```text
./PacketTracer: error while loading shared libraries: libOpenGL.so.0:
cannot open shared object file: No such file or directory
```

<figure>
  <img src="{{ '/assets/images/linux-packet-tracer/01-libopengl-manquant.jpg' | relative_url }}" alt="Terminal Ubuntu : avertissement de permission pour l'utilisateur _apt, puis erreur libOpenGL.so.0 introuvable au lancement de Packet Tracer.">
  <figcaption>Après l'installation : l'avertissement de permission pour _apt, puis l'erreur libOpenGL.so.0, la première qui m'a vraiment arrêtée.</figcaption>
</figure>

« Cannot open shared object file » ne dit pas que le paquet Cisco est corrompu. Ça dit qu'une bibliothèque du système manque. Sur Ubuntu, `libOpenGL.so.0` est fournie par le paquet `libopengl0`, souvent avec les paquets Mesa qui gèrent le rendu graphique.

## Une faute de frappe : libg11 au lieu de libgl1

J'ai d'abord mis à jour la liste des paquets, puis j'ai tenté d'installer ce que je croyais être les bons noms. J'ai écrit `libg11` au lieu de `libgl1`. La lettre l et le chiffre 1 se ressemblent beaucoup dans le terminal. `apt` a répondu tout de suite :

```text
sudo apt install libopengl0 libg11 libglx-mesa0 libxcb-xinerama0
Error: Unable to locate package libg11
```

<figure>
  <img src="{{ '/assets/images/linux-packet-tracer/02-libg11-introuvable.jpg' | relative_url }}" alt="Terminal Ubuntu : apt refuse d'installer le paquet libg11, qui n'existe pas, à la place de libgl1.">
  <figcaption>Impossible de trouver le paquet libg11 : ce nom n'existe pas. J'avais inversé un l et un 1.</figcaption>
</figure>

Le message est clair, encore faut-il le lire jusqu'au bout au lieu de relancer la même commande. Avant d'installer un paquet dont on n'est pas sûre, je vérifie maintenant le nom exact :

```bash
apt search libgl1
```

## Les paquets Mesa, puis l'erreur du greffon Qt xcb

Une fois les noms corrigés, j'ai installé les bibliothèques Mesa et les paquets liés : `libgl1-mesa-dri`, `libglx-mesa0`, `libegl-mesa0`, `mesa-libgallium`, `libgbm1`, `libxcb-xinerama0`, `mesa-vulkan-drivers`, `libopengl0`. L'erreur `libOpenGL.so.0` a disparu. Une deuxième a pris sa place :

```text
This application failed to start because no Qt platform plugin could be initialized.
Reinstalling the application may fix this problem.
Available platform plugins are: linuxfb, xcb.
Aborted (core dumped)
```

<figure>
  <img src="{{ '/assets/images/linux-packet-tracer/03-qt-xcb-core-dump.jpg' | relative_url }}" alt="Terminal Ubuntu : installation des paquets Mesa, puis échec de Packet Tracer, greffon Qt xcb non initialisé, core dumped.">
  <figcaption>Les paquets Mesa sont en place, mais Packet Tracer s'arrête encore : Qt ne peut pas initialiser son greffon xcb.</figcaption>
</figure>

Le message propose de réinstaller l'application. Ce n'était pas la bonne piste. Packet Tracer s'appuie sur Qt pour l'affichage, et `xcb` est le pont vers le serveur X11. Quand ce greffon refuse de démarrer, il manque en général d'autres bibliothèques XCB, typiquement `libxcb-cursor0` et `libxkbcommon-x11-0`, en plus de `libxcb-xinerama0` déjà installé. En installant les dépendances xcb manquantes, Packet Tracer s'est enfin lancé. Le 5 novembre à 13 h 26, j'ai eu l'écran de connexion Cisco Networking Academy.

<figure>
  <img src="{{ '/assets/images/linux-packet-tracer/04-packet-tracer-login.jpg' | relative_url }}" alt="Écran de connexion Cisco Packet Tracer, logo Cisco Networking Academy, sur mon Dell Latitude.">
  <figcaption>Packet Tracer s'ouvre enfin : l'écran de connexion Cisco Networking Academy.</figcaption>
</figure>

## Perte d'accès à ma session

Quelque temps plus tard, autre incident, plus stressant. Mon mot de passe utilisateur n'était plus accepté. La machine démarrait jusqu'à l'écran de connexion, puis je restais bloquée. Plus de session, plus d'accès à mes TP. Le système, lui, démarrait : ce n'était donc ni un disque inaccessible, ni un noyau qui refusait de charger. Le problème était au niveau du compte.

### Le menu de démarrage du Dell

J'ai ouvert le menu de démarrage ponctuel du Latitude E5250. L'écran indique clairement le mode : `Boot mode is set to: LEGACY` et `Secure Boot: OFF`. Deux familles d'entrées apparaissent : le démarrage Legacy, avec `Internal HDD` sélectionné, et le démarrage UEFI, avec une entrée Ubuntu et un disque UEFI. En bas, la révision du BIOS est A05.

<figure>
  <img src="{{ '/assets/images/linux-packet-tracer/07-dell-boot-menu-legacy.jpg' | relative_url }}" alt="Menu de démarrage ponctuel du Dell Latitude E5250 : mode Legacy, Secure Boot désactivé, Internal HDD sélectionné, BIOS A05.">
  <figcaption>Le menu de démarrage ponctuel du Dell. Mode Legacy, Secure Boot désactivé, disque interne sélectionné. Le BIOS est en révision A05.</figcaption>
</figure>

Legacy et UEFI sont deux façons, pour le micrologiciel, de lancer le système. Sur ce poste un peu ancien, les deux peuvent cohabiter dans le même menu. J'ai utilisé l'entrée du disque interne en Legacy pour arriver jusqu'à GRUB, le chargeur d'Ubuntu.

### Le menu GRUB et l'invite grub>

Le menu GRUB propose Ubuntu, les options avancées, deux tests de mémoire, et une entrée vers les réglages du micrologiciel UEFI. Pour réinitialiser un mot de passe, ce n'est pas Ubuntu en démarrage normal qu'il me fallait, mais les options avancées, puis le mode de récupération.

<figure>
  <img src="{{ '/assets/images/linux-packet-tracer/08-menu-grub.jpg' | relative_url }}" alt="Menu GRUB : Ubuntu, Advanced options for Ubuntu, tests de mémoire, UEFI Firmware Settings.">
  <figcaption>Le menu GRUB. L'entrée utile, dans mon cas, était Advanced options for Ubuntu, pour rejoindre le mode de récupération.</figcaption>
</figure>

À un moment, je suis aussi tombée sur un écran presque vide, avec seulement l'invite `grub>`. Ce n'est pas une session Ubuntu. C'est l'interpréteur minimal de GNU GRUB 2.14, un petit shell propre au chargeur de démarrage, utile pour lancer un système à la main si le menu ne s'affiche pas. Ce n'est pas là que l'on change un mot de passe utilisateur. Pour ça, il faut le menu GRUB, puis le mode de récupération.

<figure>
  <img src="{{ '/assets/images/linux-packet-tracer/09-invite-grub.jpg' | relative_url }}" alt="Écran GNU GRUB 2.14 avec l'invite minimale grub> sur mon Dell Latitude.">
  <figcaption>L'invite grub> : on est encore dans GRUB, pas dans Ubuntu. Ce n'est pas l'écran qui permet de réinitialiser le mot de passe du compte.</figcaption>
</figure>

### Le menu de récupération et le shell root

Dans le mode de récupération s'ouvre un menu intitulé `Recovery Menu (filesystem state: read-only)`. Le titre le dit : la partition racine est montée en lecture seule, pour éviter d'écrire par accident. J'ai choisi l'entrée `root`, « Drop to root shell prompt ».

<figure>
  <img src="{{ '/assets/images/linux-packet-tracer/05-recovery-menu.jpg' | relative_url }}" alt="Menu de récupération Ubuntu, système de fichiers en lecture seule, entrée root mise en surbrillance.">
  <figcaption>Le menu de récupération, système de fichiers en lecture seule. L'entrée root ouvre un interpréteur administrateur.</figcaption>
</figure>

Avant de changer quoi que ce soit, j'ai remonté la racine en lecture-écriture, puis j'ai lancé `passwd` sur mon compte, `saun` :

```bash
mount -o remount,rw /
passwd saun
```

Linux n'a pas accepté mon premier essai : `BAD PASSWORD: The password is shorter than 8 characters`. Les règles PAM exigent au moins huit caractères. À la confirmation, j'ai aussi eu `Sorry, passwords do not match` : les deux saisies n'étaient pas identiques. Au passage suivant, `passwd` a confirmé : `password updated successfully`.

Pour redémarrer, j'ai d'abord tapé `REBOOT` en majuscules. Réponse : `REBOOT: command not found`. Sous Linux, les commandes sont sensibles à la casse. La bonne commande est `reboot`, tout en minuscules.

<figure>
  <img src="{{ '/assets/images/linux-packet-tracer/06-root-shell-passwd-reboot.jpg' | relative_url }}" alt="Shell root en mode récupération : partition remontée en lecture-écriture, mot de passe trop court puis mis à jour, commande REBOOT introuvable.">
  <figcaption>Dans le shell root : remount en lecture-écriture, règles de mot de passe à huit caractères, puis l'erreur REBOOT. Il fallait écrire reboot.</figcaption>
</figure>

Après le redémarrage, j'ai pu rouvrir une session. Je n'ai pas eu à réinstaller Ubuntu.

## Une note de sécurité

Cette procédure m'a aussi rappelé une chose importante pour la suite, vers la cybersécurité. En mode de récupération, si GRUB n'est pas protégé, on obtient un interpréteur root sans saisir le mot de passe de l'utilisateur. Sur mon poste personnel, c'est précisément ce qui m'a permis de reprendre la main. Dans une entreprise, c'est un risque : une personne qui a un accès physique à la machine peut faire la même chose. On protège alors GRUB avec un mot de passe, et on chiffre le disque, par exemple avec LUKS, pour que le mode de récupération ne suffise pas à lire ou à modifier les données. L'accès physique à un poste de travail est déjà un accès à contrôler, pas seulement une astuce pour soi.

## Ce que je retiens

Je lis le message d'erreur jusqu'au bout avant de tout réinstaller. Un nom de paquet se vérifie, surtout quand un l et un 1 se ressemblent. Un avertissement `_apt` sur un `.deb` placé dans mon dossier personnel parle souvent d'un droit d'accès, pas d'un paquet raté. Quand Qt refuse le greffon `xcb`, ce sont en général des bibliothèques XCB qui manquent, pas Packet Tracer lui-même. Pour le compte, `passwd` impose une longueur minimale, et `reboot` s'écrit en minuscules. Enfin, le mode de récupération est puissant : sur un poste professionnel, il se protège, avec un mot de passe GRUB et un disque chiffré.

| Commande | Rôle |
| --- | --- |
| `sudo apt install ./CiscoPacketTracer_901_Ubuntu_64bit.deb` | Installe le paquet local de Packet Tracer |
| `sudo apt update` | Met à jour la liste des paquets |
| `apt search libgl1` | Vérifie le nom exact d'un paquet avant de l'installer |
| `sudo apt install libopengl0 libgl1 libglx-mesa0 libxcb-xinerama0` | Installe les bibliothèques graphiques souvent manquantes |
| `sudo apt install libxcb-cursor0 libxkbcommon-x11-0` | Dépendances xcb typiques pour le greffon Qt |
| `mount -o remount,rw /` | Remonte la partition racine en lecture-écriture |
| `passwd utilisateur` | Réinitialise le mot de passe d'un compte |
| `reboot` | Redémarre la machine, obligatoirement en minuscules |

<div class="english-summary">

## In English

On 5 November, I installed Cisco Packet Tracer 9.0.1 on my Dell Latitude E5250 running Ubuntu. The installer stopped on the Cisco licence: first a pager (`Press q to quit`), then a prompt where I chose `2` to accept the EULA. After the install, `apt` warned that the user `_apt` could not read the `.deb` in my home directory and fell back to an unsandboxed root read. That warning looks serious, but the real failure came when I launched the application: `libOpenGL.so.0` was missing. I then mistyped a package name, `libg11` instead of `libgl1`, and `apt` could not find it. The lesson is to read the error and check exact names with `apt search`. After installing Mesa and `libopengl0`, a second error appeared: Qt could not initialise its `xcb` plugin and the process aborted. Installing the missing xcb dependencies, typically `libxcb-cursor0` and `libxkbcommon-x11-0`, is what unblocked it. Packet Tracer then opened on the Cisco Networking Academy login screen.

I later lost access to my Ubuntu account. The system still reached the login screen, so I used the Dell one-time boot menu (Legacy mode, Secure Boot off, BIOS A05), the GRUB menu and recovery mode. I also landed on the minimal `grub>` prompt from GNU GRUB 2.14: that is GRUB's own shell, not Ubuntu, and it is not where a user password is reset. From the recovery menu I opened a root shell, remounted the root filesystem read-write, and ran `passwd`. PAM rejected a password shorter than eight characters, and a confirmation mismatch produced `Sorry, passwords do not match`, before the password was updated. Typing `REBOOT` in capitals failed: Linux commands are case-sensitive; the command is `reboot`. I did not reinstall the system.

This also has a security side. If GRUB is unprotected, recovery mode gives a root shell without the user password. On a personal laptop, that is how I got back in. In a company, GRUB should be password-protected and the disk should be encrypted, because physical access to a workstation is already an access path to control.

</div>
