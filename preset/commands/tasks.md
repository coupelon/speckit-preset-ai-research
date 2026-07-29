---
description: "Redirection : dans la methodologie recherche, le decoupage en taches n'existe pas"
---

## User Input
$ARGUMENTS

## Contexte

Ce projet utilise la methodologie de recherche systematique. Le creneau `tasks` du cycle
Spec Kit — decouper un plan en taches implementables — n'a pas d'equivalent ici. Ce qui
s'en rapproche est le classement des pistes explorees, fourni par l'extension `research`.

## Action

Ne rien produire. Afficher exactement ce message a l'utilisateur, en propageant $ARGUMENTS :

```
/speckit.tasks n'est pas utilise dans cette methodologie.

Pour evaluer, comparer et trier les pistes explorees :
  /speckit.research.dashboard                    # tableau de bord complet
  /speckit.research.dashboard L0001 L0003        # comparaison de pistes
  /speckit.research.dashboard top 5              # les 5 meilleures
  /speckit.research.dashboard gaps               # lacunes de couverture
```

Si $ARGUMENTS est non vide, proposer la commande equivalente prete a copier :
`/speckit.research.dashboard $ARGUMENTS`.

Si l'extension `research` n'est pas installee (aucune commande `/speckit.research.*`
disponible), indiquer a la place :

```
L'extension research n'est pas installee dans ce projet.
  specify extension add research
```
