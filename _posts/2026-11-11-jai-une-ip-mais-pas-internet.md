---
layout: post
title: "'J'ai une IP mais pas Internet' : la logique au service du diagnostic"
date: 2026-11-11 18:00:00 +0100
tags:
  - Réseaux
  - Cybersécurité
excerpt: "En cours, on travaille l'implication et les lois de De Morgan. J'ai voulu relier cela à un ticket très courant : le poste a une adresse IP, et pourtant il n'a pas Internet."
---

Le module de logique m'a d'abord paru loin de mon métier. Puis j'ai appliqué cette logique à un message très courant en support : « j'ai une adresse IP, mais je n'ai pas Internet ». Depuis, je m'en sers pour ne pas sauter à la mauvaise cause.

<!--more-->

Une proposition est une phrase vraie ou fausse, pas un ordre ni une question. Dans ce ticket, il y a déjà deux affirmations. P : « mon poste a une adresse IP ». Q : « mon poste a accès à Internet ». L'utilisateur dit que P est vraie et que Q est fausse. On est dans la situation `P ∧ ¬Q`. C'est un constat. Ce n'est pas encore un diagnostic.

Le piège, que j'ai commis moi-même, est d'entendre ce message comme une implication : `P → Q`, « si j'ai une adresse IP, alors j'ai Internet ». Cette implication n'est pas valable. Avoir une adresse est souvent nécessaire pour sortir sur Internet, mais ce n'est pas suffisant. L'adresse, le masque et la passerelle peuvent être en place, et l'accès peut quand même échouer : passerelle injoignable, DNS muet, filtrage, câble instable, proxy.

Pour dépanner, je découpe donc le problème en propositions plus précises. A : la configuration IP est valide. G : la passerelle répond. D : la résolution DNS fonctionne. F : un filtrage bloque le trafic sortant. Une hypothèse de travail, que je dois tester et non croire a priori, est : `(A ∧ G ∧ D ∧ ¬F) → Q`. Si Q est fausse, cette implication me dit seulement que l'une des conditions à gauche est fausse. Elle ne me dit pas laquelle.

L'implication ne se lit que dans un sens. Si `X → Y` et que Y est fausse, alors X est forcément fausse : c'est le contraposé. Si Y est vraie, on ne peut rien conclure sur X. Une règle juste serait : « si la passerelle est injoignable, alors je n'ai pas Internet », soit `¬G → ¬Q`. Le contraposé, `Q → G`, aide à vérifier un diagnostic, pas à en inventer un.

Le sophisme classique est l'affirmation du conséquent. J'ai la règle `¬G → ¬Q`. J'observe `¬Q` (pas d'Internet). J'en conclus `¬G` (la passerelle est en panne). Ce raisonnement n'est pas valide. « Pas d'Internet » peut avoir plusieurs causes ; l'observer ne désigne pas la passerelle. C'est l'erreur que l'on fait quand on change la passerelle, le câble ou le DNS au hasard, parce que « ça expliquerait le symptôme ». Oui, ça l'expliquerait. Beaucoup d'autres choses aussi. Un symptôme n'est pas une preuve de la cause.

Les lois de De Morgan m'aident à ne pas tout mélanger. `¬(G ∧ D)` équivaut à `¬G ∨ ¬D` : la passerelle est en défaut, ou le DNS, ou les deux. Cela n'autorise pas à conclure que les deux sont en défaut. Je teste chaque proposition séparément, par exemple un ping vers la passerelle, puis une résolution de nom. « Le réseau ne marche pas » n'est pas une proposition utile. Je la découpe jusqu'à ce que chaque phrase puisse être vraie ou fausse après un test unique.

La méthode que je retiens tient en quelques gestes. J'écris le ticket sous forme de propositions, sans ajouter une implication qui n'a pas été établie. Je liste les conditions qui permettraient l'accès Internet, en me souvenant que la défaillance de n'importe laquelle suffit. Je les teste une par une : configuration IP, passerelle, DNS, puis un éventuel filtrage. Je n'affirme une cause que lorsque les autres ont été écartées, et je note le résultat de chaque test pour pouvoir le transmettre.

La logique ne remplace pas les outils : `ip`, `ping`, `nslookup`, les journaux. Elle m'empêche de confondre une explication possible avec une explication démontrée. C'est ce que je veux emporter en alternance : ne pas me précipiter sur la première cause plausible.

<div class="english-summary">

## In English

In class we study propositional logic. I applied it to a common support ticket: "I have an IP address but no Internet." That message is an observation, `P ∧ ¬Q`, not a diagnosis, and it does not mean "if I have an IP address, then I have Internet." A valid address, mask and gateway can still fail because the gateway is unreachable, DNS does not answer, or a filter is blocking traffic. If Internet access requires several conditions to be true, a failure only tells me that one of them is false, not which one. I also try to avoid affirming the consequent: seeing "no Internet" does not prove that the gateway is down, even if that would explain the symptom. I test each condition separately, record whether it is true or false, and only then name a cause. Logic does not replace `ping` or `nslookup`. It stops me from treating a possible explanation as a proven one.

</div>
