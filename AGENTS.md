# Contexte projet — Recherche systematique

Ce projet utilise le bundle Spec Kit **ai-research** pour la recherche systematique iterative.
Il combine deux composants :

- l'**extension `research`** — l'activite de recherche elle-meme (exploration, checkpoints, scoring)
- le **preset `ai-research`** — la localisation des creneaux core en vocabulaire recherche

## Commandes disponibles

Creneaux du cycle Spec Kit, relus en vocabulaire recherche (preset) :

- `/speckit.constitution` — Etablir les principes du projet (interactif, une seule fois)
- `/speckit.specify` — Definir un sujet de recherche
- `/speckit.implement` — Produire le livrable final

Activite de recherche (extension) :

- `/speckit.research.explore` — Explorer les pistes (boucle iterative avec scoring)
- `/speckit.research.checkpoint` — Synthese intermediaire (auto toutes les N pistes)
- `/speckit.research.dashboard` — Evaluer, comparer et trier les pistes
- `/speckit.research.score` — Scorer ou re-evaluer une piste

`/speckit.plan` et `/speckit.tasks` ne sont pas utilises : ces deux creneaux n'ont pas
d'equivalent dans cette methodologie. Les invoquer affiche une redirection vers
`/speckit.research.explore` et `/speckit.research.dashboard`.

## Workflow

constitution → specify → research.explore (boucle) → research.checkpoint (periodique) → research.dashboard → implement

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
- Les templates et agents de l'extension sont dans `.specify/extensions/research/`
