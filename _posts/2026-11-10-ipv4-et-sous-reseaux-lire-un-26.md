---
layout: post
title: "IPv4 et sous-réseaux : lire un /26 sans paniquer"
date: 2026-11-10 18:00:00 +0100
tags:
  - Réseaux
  - Adressage IP
excerpt: "Un /26 n'est pas plus mystérieux qu'un /24. Je me suis fait une méthode courte pour retrouver le réseau, les hôtes et la diffusion, sans me tromper d'un cran."
---

Le `/24` me semblait encore raisonnable. Le `/26` me bloquait davantage : le chiffre est plus grand, le réseau est plus petit, et on se trompe vite d'un cran sur le dernier octet. J'ai posé une méthode que je peux refaire sans paniquer, même devant une interface à configurer.

<!--more-->

Une adresse IPv4 tient sur 32 bits, souvent écrite en quatre octets, par exemple `192.168.10.15`. Le masque dit où s'arrête la partie réseau et où commence la partie hôte. On peut l'écrire en décimal (`255.255.255.192`) ou en CIDR (`/26`). Le nombre après la barre, c'est simplement le nombre de bits de réseau, en partant de la gauche. Un `/26` a donc 26 bits de réseau et 6 bits d'hôte.

Le calcul des hôtes suit toujours la même règle. Avec `n` bits d'hôte, il y a `2^n` adresses au total. On en réserve deux : l'adresse de réseau (tous les bits d'hôte à 0) et l'adresse de diffusion (tous les bits d'hôte à 1). Il reste `2^n - 2` adresses utilisables. Pour un `/26`, `n` vaut 6 : 64 adresses, 62 hôtes utilisables. Le masque est `255.255.255.192`.

Si je garde `192.168.10.0/24`, j'ai un seul bloc de 254 hôtes. En `/26`, je le découpe en sous-réseaux plus petits, pour isoler un service, réduire les domaines de diffusion, et n'attribuer que ce dont on a besoin.

Voici l'exemple que je retravaille jusqu'à ce qu'il devienne automatique. Je pars de `192.168.10.0/24` et je passe en `/26`. Je « prête » 2 bits à la partie réseau (`26 - 24 = 2`). Je peux donc créer `2^2 = 4` sous-réseaux. Le pas entre deux adresses de réseau est 64, soit `2^6`, soit aussi `256 - 192`. Les quatre blocs tombent alors ainsi :

| Sous-réseau | Réseau | Premier hôte | Dernier hôte | Diffusion |
| --- | --- | --- | --- | --- |
| 1 | 192.168.10.0/26 | 192.168.10.1 | 192.168.10.62 | 192.168.10.63 |
| 2 | 192.168.10.64/26 | 192.168.10.65 | 192.168.10.126 | 192.168.10.127 |
| 3 | 192.168.10.128/26 | 192.168.10.129 | 192.168.10.190 | 192.168.10.191 |
| 4 | 192.168.10.192/26 | 192.168.10.193 | 192.168.10.254 | 192.168.10.255 |

Avant d'utiliser ce tableau, je vérifie trois choses. La diffusion d'un sous-réseau est juste avant le réseau suivant : `.63` puis `.64`. Le dernier bloc s'arrête à `.255`, qui était déjà la diffusion du `/24` d'origine. Et `4 × 64 = 256` : pas de trou, pas de chevauchement.

Pour ranger une adresse, je compare le dernier octet au pas de 64. Prenons `192.168.10.100/26`. 100 se trouve entre 64 et 127, donc l'adresse appartient à `192.168.10.64/26`. Sa diffusion est `192.168.10.127`. Elle ne peut servir ni de réseau ni de diffusion. Autre piège : `192.168.10.63` n'est pas attribuable à un poste en `/26`, parce que c'est déjà la diffusion du premier sous-réseau.

Pour ne pas perdre la main, je revois aussi le `/27` : le même `/24` donne alors 8 sous-réseaux de 30 hôtes utilisables, avec un pas de 32. Un `/28` laisse 14 hôtes.

Ce que je garde, c'est une discipline plus qu'une formule. Avant de configurer une interface, je calcule le réseau, le premier hôte, le dernier hôte et la diffusion. Une erreur d'un cran suffit à faire se chevaucher deux services, ou à placer une machine hors de sa passerelle. Le préfixe CIDR est plus rapide à relire qu'un masque en décimal, à condition de savoir passer de l'un à l'autre sans hésiter.

<div class="english-summary">

## In English

A `/26` used to make me hesitate more than a `/24`. The method I now follow is straightforward. A `/26` has 26 network bits and 6 host bits, so 64 addresses and 62 usable hosts, with the mask `255.255.255.192`. If I split `192.168.10.0/24` into `/26` networks, I borrow 2 bits and get 4 subnets, each with a step of 64: `.0`, `.64`, `.128` and `.192`. I always check that a subnet's broadcast sits just before the next network, that the last block ends at `.255`, and that `4 × 64 = 256`. To place an address such as `192.168.10.100/26`, I compare the last octet with that step: it belongs to `192.168.10.64/26`. I do this arithmetic before I configure an interface, because being off by one step is enough for two networks to overlap.

</div>
