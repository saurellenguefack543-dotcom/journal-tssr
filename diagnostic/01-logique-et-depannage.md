# Logique propositionnelle et dépannage d'un ticket réseau

# Propositional logic and troubleshooting a network ticket

Module : M306 Logique et graphes, mis en regard du dépannage réseau.
Date : novembre 2026.

En cours, on travaille la logique propositionnelle pour elle-même : propositions, connecteurs, implication, lois de De Morgan. J'ai voulu relier cela à un ticket que l'on reçoit souvent en support : « j'ai une adresse IP mais pas Internet ». Ces notes expliquent comment je m'en sers pour éviter de sauter à la mauvaise cause.

In class, we study propositional logic on its own: propositions, connectives, implication, De Morgan's laws. I wanted to connect that to a ticket we often get in support: "I have an IP address but no Internet". These notes explain how I use it to avoid jumping to the wrong cause.

## Traduire le ticket en propositions / Turning the ticket into propositions

Une proposition est une phrase qui est vraie ou fausse, pas un ordre ni une question. Le message de l'utilisateur mélange déjà deux affirmations :

A proposition is a sentence that is either true or false, not an order or a question. The user's message already mixes two claims:

- P : « mon poste a une adresse IP »
- Q : « mon poste a accès à Internet »

Le ticket dit : P est vraie, et Q est fausse. Autrement dit, on est dans la situation `P ∧ ¬Q`.

The ticket says: P is true, and Q is false. In other words, we are in the situation `P ∧ ¬Q`.

Beaucoup de gens, moi la première au début, entendent ce message comme s'il y avait une implication `P → Q` : « si j'ai une adresse IP, alors j'ai Internet ». Cette implication n'est pas valable. Avoir une adresse IP est souvent nécessaire pour sortir sur Internet, mais ce n'est pas suffisant. Une configuration IP complète (adresse, masque, passerelle) peut être en place, et l'accès Internet peut quand même échouer : passerelle injoignable, DNS qui ne répond pas, règle de filtrage, câble ou Wi-Fi instable, proxy, et ainsi de suite.

Many people, myself included at first, hear that message as if there were an implication `P → Q`: "if I have an IP address, then I have Internet". That implication does not hold. Having an IP address is often necessary to reach the Internet, but it is not sufficient. A complete IP configuration (address, mask, gateway) can be in place, and Internet access can still fail: unreachable gateway, DNS that does not answer, a filtering rule, an unstable cable or Wi-Fi link, a proxy, and so on.

Pour dépanner, j'ajoute donc d'autres propositions, plus précises :

So, to troubleshoot, I add more precise propositions:

- A : « le poste a une configuration IP valide (adresse, masque, passerelle) »
- G : « la passerelle par défaut répond »
- D : « la résolution DNS fonctionne »
- F : « un filtrage bloque le trafic sortant »

Une hypothèse de travail, que je devrai vérifier et non pas croire a priori, est :

A working hypothesis, which I must test rather than believe from the start, is:

`(A ∧ G ∧ D ∧ ¬F) → Q`

Si Q est fausse, cette implication me dit seulement que l'une des conditions à gauche est fausse. Elle ne me dit pas laquelle.

If Q is false, that implication only tells me that one of the conditions on the left is false. It does not tell me which one.

## L'implication ne se lit que dans un sens / Implication only runs one way

En logique, `X → Y` signifie : si X est vraie, alors Y est vraie. Si Y est fausse, alors X est forcément fausse (c'est le contraposé : `¬Y → ¬X`). En revanche, si Y est vraie, on ne peut rien conclure sur X. Et si X est fausse, on ne peut rien conclure sur Y.

In logic, `X → Y` means: if X is true, then Y is true. If Y is false, then X must be false (that is the contrapositive: `¬Y → ¬X`). But if Y is true, nothing follows about X. And if X is false, nothing follows about Y.

Sur le ticket, une règle juste est par exemple : « si la passerelle est injoignable, alors je n'ai pas Internet », soit `¬G → ¬Q`. Le contraposé est `Q → G` : si j'ai Internet, la passerelle est forcément joignable. Cela aide à vérifier un diagnostic, pas à en inventer un.

On this ticket, a sound rule is for example: "if the gateway is unreachable, then I have no Internet", that is `¬G → ¬Q`. The contrapositive is `Q → G`: if I have Internet, the gateway must be reachable. That helps me check a diagnosis, not invent one.

## Le sophisme de l'affirmation du conséquent / The fallacy of affirming the consequent

Le piège classique est le suivant. J'ai une règle `¬G → ¬Q`. J'observe `¬Q` (pas d'Internet). J'en conclus `¬G` (la passerelle est en panne). Ce raisonnement n'est pas valide. C'est le sophisme de l'affirmation du conséquent : on part de `X → Y`, on voit que Y est vraie, et on affirme X. Ici, « pas d'Internet » est la conséquence de plusieurs causes possibles. L'observer ne désigne pas la passerelle.

The classic trap is this. I have the rule `¬G → ¬Q`. I observe `¬Q` (no Internet). I conclude `¬G` (the gateway is down). That reasoning is not valid. It is the fallacy of affirming the consequent: one starts from `X → Y`, sees that Y is true, and asserts X. Here, "no Internet" is the consequence of several possible causes. Observing it does not point to the gateway.

C'est exactement l'erreur que l'on commet quand on change la passerelle, ou le câble, ou le serveur DNS, au hasard, parce que « ça expliquerait le symptôme ». Oui, ça l'expliquerait. Beaucoup d'autres choses aussi. Un symptôme n'est pas une preuve de la cause.

That is exactly the mistake we make when we change the gateway, or the cable, or the DNS server, at random, because "that would explain the symptom". Yes, it would. So would many other things. A symptom is not proof of the cause.

## Les lois de De Morgan / De Morgan's laws

Les lois de De Morgan disent comment nier une conjonction ou une disjonction :

De Morgan's laws say how to negate a conjunction or a disjunction:

- `¬(G ∧ D) ≡ ¬G ∨ ¬D`
- `¬(G ∨ D) ≡ ¬G ∧ ¬D`

Si je suppose que, pour avoir Internet, il faut au moins que la passerelle et le DNS fonctionnent, alors « je n'ai pas (passerelle et DNS en état) » signifie : la passerelle est en défaut, ou le DNS est en défaut, ou les deux. Cela n'autorise pas à conclure que les deux sont en défaut. Je dois tester chaque proposition séparément : un ping vers la passerelle, puis une résolution de nom, par exemple.

If I assume that, to have Internet, the gateway and DNS must at least both work, then "I do not have (gateway and DNS in working order)" means: the gateway is failing, or DNS is failing, or both. It does not allow me to conclude that both are failing. I have to test each proposition separately: a ping to the gateway, then a name resolution, for example.

De Morgan m'empêche aussi de tout regrouper dans un seul « le réseau ne marche pas ». « Le réseau ne marche pas » n'est pas une proposition utile, parce qu'elle mélange plusieurs faits. Je la découpe jusqu'à ce que chaque phrase puisse être vraie ou fausse après un test unique.

De Morgan also stops me from grouping everything into a single "the network is down". "The network is down" is not a useful proposition, because it mixes several facts. I break it down until each sentence can be true or false after a single test.

## Méthode que je retiens sur ce ticket / The method I am keeping for this ticket

1. Écrire le ticket sous forme de propositions, sans ajouter d'implication qui n'a pas été établie. `P ∧ ¬Q` est un constat, pas un diagnostic.
2. Lister les conditions qui, ensemble, permettraient Q, et me souvenir que la défaillance de n'importe laquelle suffit à expliquer `¬Q`.
3. Tester ces conditions une par une, dans un ordre simple : configuration IP, joignabilité de la passerelle, puis DNS, puis un éventuel filtrage.
4. Refuser le sophisme de l'affirmation du conséquent : un test qui « irait dans le sens du symptôme » ne prouve pas la cause tant que les autres causes n'ont pas été écartées.
5. Noter le résultat de chaque test comme une proposition vraie ou fausse, pour pouvoir relire le raisonnement et le transmettre.

1. Write the ticket as propositions, without adding an implication that has not been established. `P ∧ ¬Q` is an observation, not a diagnosis.
2. List the conditions that, together, would allow Q, and remember that the failure of any one of them is enough to explain `¬Q`.
3. Test those conditions one by one, in a simple order: IP configuration, gateway reachability, then DNS, then possible filtering.
4. Reject the fallacy of affirming the consequent: a test that "fits the symptom" does not prove the cause until the other causes have been ruled out.
5. Record the result of each test as a true or false proposition, so that the reasoning can be reread and handed over.

C'est ce que je retiens du module M306 pour le support : la logique ne remplace pas les outils (`ip`, `ping`, `nslookup`, les journaux). Elle m'empêche de confondre une explication possible avec une explication démontrée.

That is what I am taking from the M306 module into support work: logic does not replace the tools (`ip`, `ping`, `nslookup`, the logs). It stops me from confusing a possible explanation with a proven one.
