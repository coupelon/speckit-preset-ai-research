---
description: "Redirection : dans la methodologie recherche, planifier c'est explorer"
---

## User Input
$ARGUMENTS

## Contexte

Ce projet utilise la methodologie de recherche systematique. Le creneau `plan` du cycle
Spec Kit — decider *comment* construire une fonctionnalite — n'existe pas ici : le travail
de terrain est l'exploration iterative de pistes, fournie par l'extension `research`.

## Action

Ne rien produire. Afficher exactement ce message a l'utilisateur, en propageant $ARGUMENTS :

```
/speckit.plan n'est pas utilise dans cette methodologie.

Le travail d'exploration se lance avec :
  /speckit.research.explore            # prochaine piste pending
  /speckit.research.explore 5          # les 5 prochaines pistes
  /speckit.research.explore L0042      # une piste precise

Vue d'ensemble des pistes deja explorees :
  /speckit.research.dashboard
```

Si $ARGUMENTS est non vide, proposer la commande equivalente prete a copier :
`/speckit.research.explore $ARGUMENTS`.

Si l'extension `research` n'est pas installee (aucune commande `/speckit.research.*`
disponible), indiquer a la place :

```
L'extension research n'est pas installee dans ce projet.
  specify extension add research
```
