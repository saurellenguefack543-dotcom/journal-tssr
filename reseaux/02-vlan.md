# VLAN : ports d'accès, trunks et 802.1Q / VLANs: access ports, trunks and 802.1Q

Module : M301 Réseaux IP, et révision du CCNA Introduction to Networks.
Date : novembre 2026.

## Ce qu'est un VLAN / What a VLAN is

Un VLAN (Virtual Local Area Network) est un réseau local logique défini sur un commutateur, indépendamment des câbles physiques. Les ports d'un même VLAN se comportent comme s'ils étaient sur le même commutateur dédié : ils partagent le même domaine de diffusion. Deux postes placés dans des VLAN différents ne communiquent pas directement au niveau 2, même s'ils sont branchés sur le même équipement.

On crée des VLAN pour plusieurs raisons pratiques : séparer un service (direction, pédagogie, invités, imprimantes), réduire la taille des domaines de diffusion, et poser une première limite de sécurité. Ce n'est pas un pare-feu, mais cela évite qu'un trafic inutile ou indésirable circule partout.

A VLAN (Virtual Local Area Network) is a logical LAN defined on a switch, independently of the physical cabling. Ports in the same VLAN behave as if they sat on a dedicated switch: they share the same broadcast domain. Two hosts in different VLANs do not communicate directly at layer 2, even when they are plugged into the same device.

VLANs are created for practical reasons: to separate a service (management, teaching, guests, printers), to keep broadcast domains smaller, and to set a first security boundary. A VLAN is not a firewall, but it does stop unnecessary or unwanted traffic from flooding everywhere.

## Port d'accès et port trunk / Access ports and trunk ports

Un port d'accès (access) appartient à un seul VLAN. On y branche typiquement un poste, une imprimante ou un téléphone. La trame Ethernet qui part vers l'équipement final n'est pas étiquetée : le poste n'a pas besoin de « connaître » le numéro de VLAN.

Un port trunk transporte plusieurs VLAN à la fois, en général entre deux commutateurs, ou entre un commutateur et un routeur. Pour que le commutateur d'en face sache à quel VLAN appartient chaque trame, celle-ci est étiquetée.

An access port belongs to a single VLAN. This is where a workstation, a printer or a phone is usually connected. The Ethernet frame sent to the end device is untagged: the host does not need to know the VLAN number.

A trunk port carries several VLANs at once, typically between two switches, or between a switch and a router. So that the device at the other end can tell which VLAN a frame belongs to, the frame is tagged.

## 802.1Q

La norme IEEE 802.1Q décrit cette étiquette. Elle insère un champ de 4 octets dans l'en-tête Ethernet, juste après les adresses MAC. Ce champ contient notamment l'identifiant de VLAN (VID), sur 12 bits, donc des valeurs de 1 à 4094 en pratique (0 et 4095 sont réservés).

Sur le trunk, un VLAN peut rester non étiqueté : c'est le VLAN natif. Les deux extrémités du trunk doivent s'accorder sur le même VLAN natif, sinon des trames passent dans le mauvais réseau. Je retiens de toujours le documenter, et de ne pas le laisser par habitude sur le VLAN 1 si je peux l'éviter.

The IEEE 802.1Q standard describes this tag. It inserts a 4-byte field into the Ethernet header, right after the MAC addresses. That field includes the VLAN identifier (VID), on 12 bits, so values from 1 to 4094 in practice (0 and 4095 are reserved).

On a trunk, one VLAN may remain untagged: the native VLAN. Both ends of the trunk must agree on the same native VLAN, otherwise frames leak into the wrong network. I make a point of documenting it, and of not leaving it on VLAN 1 out of habit when I can avoid it.

## Exemple de configuration Cisco IOS / Sample Cisco IOS configuration

Exemple générique, sur un commutateur : le VLAN 10 pour la direction, le VLAN 20 pour la pédagogie, un poste sur le port `FastEthernet0/1`, et un trunk vers un autre commutateur sur `FastEthernet0/24`.

A generic example on a switch: VLAN 10 for management, VLAN 20 for teaching, a host on `FastEthernet0/1`, and a trunk to another switch on `FastEthernet0/24`.

```text
enable
configure terminal

vlan 10
 name DIRECTION
vlan 20
 name PEDAGOGIE
exit

interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
 no shutdown
exit

interface FastEthernet0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
end
```

Sur beaucoup de commutateurs récents, 802.1Q est le seul encapsulage possible, donc la commande `switchport trunk encapsulation dot1q` n'existe pas, ou n'est pas nécessaire. Sur d'anciens modèles, il faut la saisir avant de passer le port en trunk. Je vérifie toujours avec `show vlan brief`, `show interfaces FastEthernet0/1 switchport` et `show interfaces trunk`.

On many recent switches, 802.1Q is the only encapsulation available, so `switchport trunk encapsulation dot1q` is either missing or unnecessary. On older models, it must be entered before the port is set to trunk. I always check with `show vlan brief`, `show interfaces FastEthernet0/1 switchport` and `show interfaces trunk`.

## Ce que je retiens / What I am keeping from this

Avant de chercher une panne de connectivité, je me demande si les deux postes sont dans le même VLAN, si le port est bien en mode access avec le bon numéro, et si le trunk autorise ce VLAN. Un poste peut avoir une adresse IP correcte et ne rien joindre, simplement parce que son port est resté dans le VLAN 1 par défaut. Le même réflexe vaut pour le VLAN natif des deux côtés d'un trunk.

Before I chase a connectivity failure, I ask whether both hosts are in the same VLAN, whether the port is truly in access mode with the right VLAN number, and whether the trunk allows that VLAN. A host can have a correct IP address and still reach nothing, simply because its port was left in the default VLAN 1. The same habit applies to the native VLAN on both ends of a trunk.
