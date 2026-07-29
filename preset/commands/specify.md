---
description: "Definir un sujet de recherche : probleme, questions, criteres de succes, perimetre"
---

## User Input
$ARGUMENTS

## Contexte

Tu definis un nouveau sujet de recherche. Cette etape est l'equivalent du "Specify" de Spec Kit : elle etablit le QUOI et le POURQUOI sans prescrire le COMMENT.

## Pre-requis

- `.specify/memory/constitution.md` doit exister (principes etablis)

## Instructions

1. Si l'utilisateur a fourni un sujet dans $ARGUMENTS, l'utiliser comme point de depart
2. Sinon, demander le sujet de recherche
3. Scanner les specs existantes dans `specs/` pour attribuer le prochain numero (001, 002...)
4. Creer le repertoire `specs/###-slug-du-sujet/`
5. Generer `specs/###-slug-du-sujet/spec.md` en suivant le template

## Template de spec.md

Le fichier `spec.md` doit contenir :

### Probleme
- Enonce clair du probleme a resoudre
- Contexte et motivation
- Impact attendu de la recherche

### Questions de recherche
Pour chaque question (Q1, Q2, Q3...) :
- Enonce de la question
- Priorite (P1 = critique, P2 = importante, P3 = exploratoire)
- Indicateurs de succes au format Given/When/Then :
  ```
  GIVEN [contexte de la recherche]
  WHEN [condition de completion]
  THEN [resultat attendu mesurable]
  ```

### Perimetre
- Inclus : domaines, periodes, geographies, langues
- Exclus : ce qui est explicitement hors perimetre
- Contraintes : temps, ressources, acces

### Axes de recherche
Pour chaque axe (academique, industriel, public) :
- Pertinence de l'axe pour ce sujet (oui/non/partiel)
- Questions specifiques a cet axe
- Sources privilegiees

### Livrables attendus
- Type de livrable : article de synthese, etat de l'art, demonstrateur, outil, rapport
- Public cible
- Format attendu

### Criteres de succes
Liste de criteres mesurables. Exemples :
- "Au moins 3 approches differentes documentees et comparees"
- "Au moins 1 demonstrateur identifie et teste"
- "H-index moyen des sources > 15"

## Initialisation des fichiers du sujet

Creer dans `specs/###-slug-du-sujet/` :

1. `spec.md` — Definition ci-dessus
2. `leads.json` — Base de pistes vide :
```json
{
  "metadata": {
    "research_id": "###",
    "topic": "[slug]",
    "created_at": "[ISO date]",
    "last_checkpoint": null,
    "checkpoint_number": 0,
    "total_explored": 0,
    "next_checkpoint_at": 10,
    "checkpoint_interval": 10,
    "confidence_threshold": 60
  },
  "statistics": {
    "by_status": { "pending": 0, "in_progress": 0, "explored": 0, "abandoned": 0 },
    "by_axis": {
      "academic": { "explored": 0, "pending": 0, "avg_score": 0 },
      "industry": { "explored": 0, "pending": 0, "avg_score": 0 },
      "public": { "explored": 0, "pending": 0, "avg_score": 0 }
    }
  },
  "leads": []
}
```
3. `ontology.md` — Squelette avec les axes identifies
4. `references.bib` — Fichier BibLaTeX vide avec en-tete
5. Repertoires : `leads/`, `sources/academic/`, `sources/industry/`, `sources/public/`, `synthesis/`

## Prochaine etape

Suggerer : `/speckit.research.explore` pour commencer l'exploration des pistes.
