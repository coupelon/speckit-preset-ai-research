# Contexte projet — Recherche systematique

Ce projet utilise le preset Spec Kit **research-methodology** pour la recherche systematique iterative.

## Commandes disponibles

- `/speckit.constitution` — Etablir les principes du projet (interactif, une seule fois)
- `/speckit.specify` — Definir un sujet de recherche
- `/speckit.plan` — Explorer les pistes (boucle iterative)
- `/speckit.tasks` — Evaluer et comparer les pistes
- `/speckit.implement` — Produire le livrable final
- `/speckit.research.checkpoint` — Synthese intermediaire (auto toutes les N pistes)
- `/speckit.research.score` — Scorer ou re-evaluer une piste

## Workflow

constitution → specify → plan (boucle) → checkpoint (periodique) → tasks → implement

## Routage des outils MCP

IMPORTANT : chaque outil a un role precis. Ne pas les interchanger.

| Besoin | Outil a utiliser | NE PAS utiliser |
|--------|-----------------|-----------------|
| Chercher un article academique | `paper-search` | `web-fetch`, `web-search` |
| Telecharger le contenu d'un article | `paper-search` (Sci-Hub) | `web-fetch` |
| Metriques auteurs (h-index, citations) | `openalex` | `paper-search` |
| Preprints ArXiv | `arxiv` | `web-fetch` |
| Chercher une source web (industrielle, publique) | `web-search` ou `searxng` | `paper-search` |
| Lire le contenu d'une page web (URL connue) | `web-fetch` | — |

## Conventions

- IDs de pistes : `LXXXX` (ex: L0001)
- IDs de questions : `Qn` (ex: Q1)
- Dates : ISO 8601
- Cles BibLaTeX : `auteur-annee-motcle`
- Les principes de scoring et de qualite sont dans `.specify/memory/constitution.md`
