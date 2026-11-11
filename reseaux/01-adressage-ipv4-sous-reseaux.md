# Adressage IPv4 et sous-réseaux / IPv4 addressing and subnetting

Module : M301 Réseaux IP, et révision du CCNA Introduction to Networks.
Date : novembre 2026.

Ces notes résument ce que j'ai compris, avec un exemple calculé en entier. Ce n'est pas un cours recopié.

These notes summarise what I have understood, with one fully worked example. This is not a copied lecture.

## À quoi sert une adresse IPv4 / What an IPv4 address is for

Une adresse IPv4 identifie une interface sur un réseau. Elle est écrite sur 32 bits, souvent présentés en quatre octets en décimal, par exemple `192.168.10.15`. Tous les appareils d'un même réseau IP doivent partager la même partie réseau, et chaque hôte doit avoir une partie hôte unique.

Pour séparer la partie réseau et la partie hôte, on utilise un masque de sous-réseau. Le masque est aussi un nombre de 32 bits : les bits à 1 marquent la partie réseau, les bits à 0 la partie hôte.

An IPv4 address identifies an interface on a network. It is 32 bits long, usually written as four decimal octets, for example `192.168.10.15`. Every device on the same IP network must share the same network part, and each host must have a unique host part.

A subnet mask splits the two parts. The mask is also 32 bits: bits set to 1 mark the network portion, bits set to 0 mark the host portion.

## Masque et notation CIDR / Masks and CIDR notation

Le même masque peut s'écrire de deux façons :

- en décimal pointé : `255.255.255.0`
- en CIDR, avec un préfixe : `/24`

Le nombre après la barre indique combien de bits, en partant de la gauche, appartiennent au réseau. Ainsi, `/24` signifie 24 bits de réseau et 8 bits d'hôte. `/26` signifie 26 bits de réseau et 6 bits d'hôte.

Quelques masques que je revois souvent :

| Préfixe CIDR | Masque | Bits d'hôte | Adresses au total | Hôtes utilisables |
| --- | --- | --- | --- | --- |
| /24 | 255.255.255.0 | 8 | 256 | 254 |
| /25 | 255.255.255.128 | 7 | 128 | 126 |
| /26 | 255.255.255.192 | 6 | 64 | 62 |
| /27 | 255.255.255.224 | 5 | 32 | 30 |
| /28 | 255.255.255.240 | 4 | 16 | 14 |
| /30 | 255.255.255.252 | 2 | 4 | 2 |

Le nombre total d'adresses d'un réseau est `2^n`, où `n` est le nombre de bits d'hôte. On retire en général deux adresses : l'adresse de réseau (tous les bits d'hôte à 0) et l'adresse de diffusion (tous les bits d'hôte à 1). Il reste donc `2^n - 2` hôtes utilisables. Un `/31` ou un `/32` suit d'autres règles (liaison point à point, adresse de boucle ou d'interface) ; je ne les utilise pas dans cet exemple.

The same mask can be written in two ways:

- dotted decimal: `255.255.255.0`
- CIDR prefix: `/24`

The number after the slash is how many bits, from the left, belong to the network. So `/24` means 24 network bits and 8 host bits. `/26` means 26 network bits and 6 host bits.

The total number of addresses in a network is `2^n`, where `n` is the number of host bits. Two addresses are usually reserved: the network address (all host bits set to 0) and the broadcast address (all host bits set to 1). That leaves `2^n - 2` usable host addresses. A `/31` or a `/32` follows other rules (point-to-point links, loopback or interface addresses); I do not use them in this example.

## Pourquoi découper un réseau / Why we subnet

Si j'ai le réseau `192.168.10.0/24`, j'ai un seul bloc de 254 hôtes utilisables. En passant à un préfixe plus long, par exemple `/26`, je découpe ce bloc en plusieurs sous-réseaux plus petits. Cela permet d'isoler des services ou des étages, de limiter la taille des domaines de diffusion, et de n'attribuer que le nombre d'adresses dont un service a réellement besoin.

If I have the network `192.168.10.0/24`, I have a single block of 254 usable hosts. By moving to a longer prefix, for example `/26`, I split that block into several smaller subnets. That makes it possible to isolate services or floors, to keep broadcast domains smaller, and to allocate only as many addresses as a service actually needs.

## Exemple travaillé : `192.168.10.0/24` découpé en `/26`

### Données de départ / Starting point

- Réseau d'origine : `192.168.10.0/24`
- Masque d'origine : `255.255.255.0`
- Nouveau préfixe : `/26`
- Nouveau masque : `255.255.255.192`

### Calculs / Arithmetic

Je prête 2 bits à la partie réseau (`26 - 24 = 2`). Je peux donc créer `2^2 = 4` sous-réseaux.

Il reste 6 bits d'hôte, donc `2^6 = 64` adresses par sous-réseau, dont `64 - 2 = 62` hôtes utilisables.

Le pas entre deux adresses de réseau est 64. On le retrouve aussi par `256 - 192 = 64`, en regardant le dernier octet du masque.

I borrow 2 bits for the network portion (`26 - 24 = 2`). That gives `2^2 = 4` subnets.

6 host bits remain, so `2^6 = 64` addresses per subnet, of which `64 - 2 = 62` are usable hosts.

The step between two network addresses is 64. The same number appears as `256 - 192 = 64`, from the last octet of the mask.

### Tableau des quatre sous-réseaux / The four subnets

| Sous-réseau | Adresse de réseau | Premier hôte | Dernier hôte | Diffusion | Hôtes utilisables |
| --- | --- | --- | --- | --- | --- |
| 1 | 192.168.10.0/26 | 192.168.10.1 | 192.168.10.62 | 192.168.10.63 | 62 |
| 2 | 192.168.10.64/26 | 192.168.10.65 | 192.168.10.126 | 192.168.10.127 | 62 |
| 3 | 192.168.10.128/26 | 192.168.10.129 | 192.168.10.190 | 192.168.10.191 | 62 |
| 4 | 192.168.10.192/26 | 192.168.10.193 | 192.168.10.254 | 192.168.10.255 | 62 |

Vérifications que je refais à chaque fois :

- le premier hôte est l'adresse de réseau plus 1 ;
- le dernier hôte est l'adresse de diffusion moins 1 ;
- la diffusion du premier sous-réseau (`192.168.10.63`) est juste avant le réseau suivant (`192.168.10.64`) ;
- le dernier sous-réseau s'arrête bien à `192.168.10.255`, qui était déjà la diffusion du `/24` d'origine ;
- `4 × 64 = 256` : je n'ai ni trou ni chevauchement dans le `/24`.

Checks I redo every time:

- the first host is the network address plus 1;
- the last host is the broadcast address minus 1;
- the first subnet's broadcast (`192.168.10.63`) sits immediately before the next network (`192.168.10.64`);
- the last subnet ends at `192.168.10.255`, which was already the broadcast of the original `/24`;
- `4 × 64 = 256`: there is no gap and no overlap inside the `/24`.

### Où se place une adresse / Where an address belongs

Pour ranger une adresse dans son sous-réseau, je compare le dernier octet au pas de 64. Exemple : `192.168.10.100/26`. 100 est compris entre 64 et 127, donc l'adresse appartient à `192.168.10.64/26`. Sa diffusion est `192.168.10.127`. Elle ne peut pas être utilisée comme adresse de réseau ni comme diffusion.

To place an address in its subnet, I compare the last octet with the step of 64. Example: `192.168.10.100/26`. 100 sits between 64 and 127, so the address belongs to `192.168.10.64/26`. Its broadcast is `192.168.10.127`. It cannot be used as a network or broadcast address.

## Exercices d'entraînement / Practice

J'ai d'abord rempli le tableau sans regarder les réponses, puis je les ai vérifiées.

I first filled the table without looking at the answers, then checked them.

| Question | Réponse |
| --- | --- |
| Combien de sous-réseaux et d'hôtes utilisables si je découpe `192.168.10.0/24` en `/27` ? | 8 sous-réseaux, 30 hôtes utilisables chacun (pas de 32 : `.0`, `.32`, `.64`, `.96`, `.128`, `.160`, `.192`, `.224`) |
| Quelle est l'adresse de diffusion de `192.168.10.80/26` ? | `192.168.10.127` (80 est dans le sous-réseau `.64`) |
| `192.168.10.63` peut-elle être attribuée à un poste en `/26` ? | Non : c'est la diffusion du premier sous-réseau |
| Combien d'hôtes utilisables dans un `/28` ? | 14 |
| How many subnets and usable hosts if I split `192.168.10.0/24` into `/27`? | 8 subnets, 30 usable hosts each (step of 32: `.0`, `.32`, `.64`, `.96`, `.128`, `.160`, `.192`, `.224`) |
| What is the broadcast address of `192.168.10.80/26`? | `192.168.10.127` (80 belongs to the `.64` subnet) |
| Can `192.168.10.63` be assigned to a host in a `/26`? | No: it is the broadcast of the first subnet |
| How many usable hosts in a `/28`? | 14 |

## Ce que je retiens / What I am keeping from this

Avant de configurer une interface, je calcule toujours le réseau, le premier hôte, le dernier hôte et la diffusion. Une erreur d'un cran sur le masque suffit à faire se chevaucher deux services, ou à placer une machine en dehors de sa passerelle. Le préfixe CIDR est plus rapide à relire qu'un masque en décimal, à condition d'avoir le réflexe de traduire l'un vers l'autre.

Before I configure an interface, I always work out the network, the first host, the last host and the broadcast. Being off by one step on the mask is enough for two services to overlap, or for a host to sit outside its gateway. The CIDR prefix is faster to read than a dotted mask, provided I can convert one into the other without hesitation.
