---
description: "Evaluer, comparer et trier les pistes par score de confiance"
---

## User Input
$ARGUMENTS

## Contexte

Le tableau de bord evalue et compare les pistes explorees. C'est l'etape qui transforme une
liste brute de pistes en un classement actionnable — pas un decoupage en taches : le creneau
`/speckit.tasks` du cycle Spec Kit n'a pas d'equivalent dans cette methodologie.

## Pre-requis

CRITICAL : Verifier que les fichiers suivants existent dans le repertoire du sujet courant (`specs/###-slug/`) :
- `spec.md` (sujet defini)
- `leads.json` (avec au moins 1 piste exploree)

## Workflow

### Mode 1 : Tableau de bord (par defaut, sans arguments)

Suivre la structure de `.specify/extensions/research/templates/dashboard-template.md`.
Generer un tableau recapitulatif de toutes les pistes explorees :

```
| ID    | Titre           | Axe        | Score | Statut   |
|-------|-----------------|------------|-------|----------|
| L0001 | [Titre court]   | academique | 85    | explored |
| L0002 | [Titre court]   | industriel | 72    | explored |
```

Ajouter les statistiques :
- Repartition par axe (academique/industriel/public)
- Score moyen par axe
- Couverture des questions de recherche de spec.md
- Pistes au-dessus/en-dessous du seuil de confiance

### Mode 2 : Comparaison (argument = liste d'IDs)

Si $ARGUMENTS contient des IDs (ex: "L0001 L0003 L0007") :

Pour chaque paire de pistes :
1. Comparer les scores critere par critere
2. Identifier les complementarites (une piste forte en theorie + une forte en pratique)
3. Detecter les contradictions entre les approches
4. Evaluer la couverture combinee des questions de recherche

Produire une matrice de comparaison :

```
| Critere           | L0001 | L0003 | L0007 |
|-------------------|-------|-------|-------|
| Serieux (/20)     | 18    | 12    | 15    |
| Popularite (/20)  | 16    | 8     | 19    |
| Applicabilite (/25)| 20   | 22    | 15    |
| Recence (/15)     | 12    | 14    | 10    |
| Replicabilite (/20)| 15   | 18    | 12    |
| **Total (/100)**  | **81**| **74**| **71**|
```

### Mode 3 : Filtrage (argument = critere)

Si $ARGUMENTS contient un filtre :
- `"top N"` : les N meilleures pistes par score total
- `"axe:academique"` : pistes d'un axe specifique
- `"question:Q1"` : pistes couvrant une question de recherche
- `"seuil:75"` : pistes au-dessus d'un score donne
- `"recent"` : pistes triees par date de publication

### Mode 4 : Analyse de gaps

Si $ARGUMENTS contient "gaps" :

1. Lire les questions de recherche dans spec.md
2. Pour chaque question, lister les pistes qui y repondent
3. Identifier les questions mal couvertes
4. Identifier les axes sous-representes
5. Suggerer des requetes MCP pour combler les lacunes :
   - `paper-search` pour les gaps academiques
   - `web-search`/`searxng` pour les gaps industriels/publics

## Sortie

```
=== Evaluation des pistes — [Titre du sujet] ===

Pistes explorees : N | Score moyen : XX/100
Axes : academique (X) | industriel (Y) | public (Z)
Au-dessus du seuil (60) : N | En-dessous : M

[Tableau ou comparaison selon le mode]

Gaps identifies :
- Q2 : 1 seule piste, score moyen 55 → explorer davantage
- Axe public : 0 pistes → lancer web-search/searxng
```

## Prochaine etape

- Gaps identifies → `/speckit.research.explore` pour explorer les lacunes
- Pistes suffisantes → `/speckit.implement` pour produire le livrable
- Re-evaluer une piste → `/speckit.research.score L0042`
- Synthese intermediaire → `/speckit.research.checkpoint`
