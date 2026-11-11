---
layout: post
title: "Les VLAN, access et trunk 802.1Q"
date: 2026-11-11 09:00:00 +0100
tags:
  - Réseaux
excerpt: "Deux postes branchés sur le même commutateur peuvent ne pas se voir. J'ai clarifié ce qu'est un VLAN, la différence entre un port d'accès et un trunk, et le rôle de l'étiquette 802.1Q."
---

En cours, on revient souvent sur une situation trompeuse : deux machines sont branchées sur le même commutateur, leurs adresses IP ont l'air correctes, et pourtant elles ne communiquent pas. Avant de suspecter le câble ou le masque, je me demande maintenant si elles sont dans le même VLAN.

<!--more-->

Un VLAN, Virtual Local Area Network, est un réseau local logique défini sur un commutateur, indépendamment du câblage physique. Les ports d'un même VLAN se comportent comme s'ils étaient sur un commutateur dédié : ils partagent le même domaine de diffusion. Deux postes placés dans des VLAN différents ne se parlent pas directement au niveau 2, même s'ils sont sur le même équipement.

On crée des VLAN pour des raisons très concrètes : séparer la direction, la pédagogie, les invités ou les imprimantes, limiter la taille des domaines de diffusion, et poser une première limite. Ce n'est pas un pare-feu. Cela empêche simplement qu'un trafic inutile, ou indésirable, circule partout.

Il faut ensuite distinguer deux types de ports. Un port d'accès, ou access, appartient à un seul VLAN. On y branche un poste, une imprimante, un téléphone. La trame envoyée vers l'équipement final n'est pas étiquetée : le poste n'a pas besoin de connaître le numéro de VLAN. Un port trunk, lui, transporte plusieurs VLAN à la fois, en général entre deux commutateurs, ou entre un commutateur et un routeur. Pour que le voisin sache à quel VLAN appartient chaque trame, celle-ci est étiquetée.

C'est le rôle de la norme IEEE 802.1Q. Elle insère un champ de 4 octets dans l'en-tête Ethernet, juste après les adresses MAC. Ce champ contient notamment l'identifiant de VLAN, le VID, sur 12 bits. En pratique, on utilise les valeurs 1 à 4094 ; 0 et 4095 sont réservés. Sur le trunk, un VLAN peut rester non étiqueté : c'est le VLAN natif. Les deux extrémités doivent s'accorder sur le même VLAN natif, sinon des trames passent dans le mauvais réseau. Je me suis promis de toujours le documenter, et de ne pas le laisser par habitude sur le VLAN 1 quand je peux l'éviter.

Pour ancrer tout cela, je me sers d'un petit exemple Cisco IOS. Le VLAN 10 sert à l'administration, le VLAN 30 aux invités. Un poste est sur `GigabitEthernet0/1`, le trunk vers l'autre commutateur est sur `GigabitEthernet0/24`.

```text
vlan 10
 name ADMIN
vlan 30
 name INVITES

interface GigabitEthernet0/1
 switchport mode access
 switchport access vlan 10

interface GigabitEthernet0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,30
```

Sur beaucoup de commutateurs récents, 802.1Q est le seul encapsulage possible, donc `switchport trunk encapsulation dot1q` n'existe pas, ou n'est pas nécessaire. Sur d'anciens modèles, il faut la saisir avant de passer le port en trunk. Je vérifie ensuite avec `show vlan brief`, `show interfaces GigabitEthernet0/1 switchport` et `show interfaces trunk`. Si le port d'accès est resté dans le VLAN 1 par défaut, le poste peut avoir une adresse IP tout à fait correcte et ne rien joindre. C'est précisément le piège dont je parlais au début.

Avant de chercher une panne de connectivité, je pose donc trois questions. Les deux postes sont-ils dans le même VLAN ? Le port est-il bien en mode access, avec le bon numéro ? Le trunk autorise-t-il ce VLAN, et le VLAN natif est-il le même des deux côtés ? Si je réponds à cela d'abord, je gagne souvent du temps.

<div class="english-summary">

## In English

A VLAN is a logical LAN on a switch. Hosts in the same VLAN share a broadcast domain; hosts in different VLANs do not talk directly at layer 2, even on the same device. An access port belongs to one VLAN and sends untagged frames to the end device. A trunk carries several VLANs at once and tags frames so the neighbour can tell them apart, using IEEE 802.1Q. Both ends of a trunk must agree on the native VLAN, and I try not to leave it on VLAN 1 out of habit. A host can have a perfectly valid IP address and still reach nothing if its port was left in the default VLAN. Before I chase a cabling or subnet problem, I now check the VLAN, the access mode, and the trunk allow list.

</div>
