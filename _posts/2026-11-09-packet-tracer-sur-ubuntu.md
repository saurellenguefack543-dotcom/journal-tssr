---
layout: post
title: "Packet Tracer sur Ubuntu : deux erreurs et une leçon"
date: 2026-11-09 18:00:00 +0100
tags:
  - Linux
excerpt: "Sur mon Dell Latitude, Packet Tracer refusait de s'ouvrir, puis je n'arrivais plus à ouvrir une session Ubuntu. Deux blocages, une même habitude : lire le message avant de tout réinstaller."
---

J'utilise un Dell Latitude E5250 sous Ubuntu pour mes travaux pratiques de réseaux. J'y installe Cisco Packet Tracer 9.0.1. Sur le papier, c'est simple. Sur cette machine, ça ne l'a pas été.

<!--more-->

Après l'installation, le logiciel ne s'ouvrait pas du tout. Pas de fenêtre, pas d'interface. Juste une ligne dans le terminal : `libOpenGL.so.0: cannot open shared object file: No such file or directory`. Il y a quelques mois, j'aurais sans doute désinstallé Packet Tracer pour le remettre proprement. Cette fois, j'ai d'abord relu le message.

« Cannot open shared object file » ne dit pas que le paquet Cisco est corrompu. Ça dit qu'une bibliothèque du système manque. Le binaire cherche `libOpenGL.so.0` et ne la trouve pas. Sur Ubuntu, ce fichier est fourni par le paquet `libopengl0`, souvent avec les paquets Mesa qui gèrent le rendu graphique. J'ai donc installé `libopengl0`, `libgl1-mesa-dri`, `libglx-mesa0` et `libglu1-mesa`.

L'erreur OpenGL a disparu. Une deuxième a pris sa place : Qt n'arrivait pas à initialiser son greffon `xcb`. Packet Tracer s'appuie sur Qt pour l'affichage, et `xcb` est le pont vers le serveur X11. Quand ce greffon refuse de démarrer, il manque en général des bibliothèques XCB, typiquement `libxcb-xinerama0`, `libxcb-cursor0` et `libxkbcommon-x11-0`. Une fois ces paquets en place, Packet Tracer s'est enfin lancé.

Je retiens une chose simple : un programme graphique qui ne s'ouvre pas n'est pas forcément mal installé. Ubuntu, surtout sur un poste un peu ancien, n'installe pas toujours les dépendances que Packet Tracer attend. Le message exact m'a évité de tout recommencer.

Quelques jours plus tard, autre incident, plus stressant. Mon mot de passe utilisateur n'était plus accepté. La machine démarrait jusqu'à l'écran de connexion, puis je restais bloquée. Plus de session, plus d'accès à mes TP. Le système, lui, démarrait : ce n'était donc ni un disque inaccessible, ni un noyau qui refusait de charger. Le problème était au niveau du compte.

J'ai redémarré, ouvert le menu GRUB, choisi les options avancées puis le mode de récupération, et ouvert un interpréteur root. Dans ce mode, la partition racine est souvent montée en lecture seule, pour éviter d'écrire par accident. Avant de changer le mot de passe, je l'ai remontée en lecture-écriture :

```bash
mount -o remount,rw /
passwd mon_utilisateur
reboot
```

Après le redémarrage, j'ai pu rouvrir une session. Je n'ai pas eu à réinstaller Ubuntu.

Deux précautions restent. Il faut vraiment remonter la racine en lecture-écriture avant `passwd`, sinon la commande échoue sans que l'on comprenne tout de suite pourquoi. Et cette procédure montre aussi qu'un accès physique à une machine donne déjà un moyen de reprendre la main. Sur un poste professionnel, ce n'est pas anodin : c'est un accès à protéger, pas seulement une astuce pour soi.

Au bout du compte, les deux incidents m'ont appris la même habitude. Je lis le symptôme jusqu'au bout, je pose ce qui fonctionne encore, et je n'installe ni ne réinstalle au hasard. C'est cette méthode que je veux garder pour l'alternance.

<div class="english-summary">

## In English

On my Dell Latitude running Ubuntu, Cisco Packet Tracer 9.0.1 would not start. The first error pointed to a missing `libOpenGL.so.0` library, which I fixed by installing `libopengl0` and the Mesa packages. A second error then appeared: Qt could not initialise its `xcb` plugin. Installing the missing XCB libraries solved it. I later lost access to my Ubuntu account. Because the system still booted, I used GRUB recovery mode, remounted the root filesystem read-write, reset the password with `passwd`, and signed back in. I did not reinstall the system. The habit I am keeping is to read the exact error and check what still works, instead of reinstalling at random. Physical access to a machine is also an access path that needs protecting.

</div>
