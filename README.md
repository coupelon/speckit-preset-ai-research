# AI Research — Preset Spec Kit

Preset pour [GitHub Spec Kit](https://github.com/github/spec-kit) qui transforme le workflow spec-driven en un framework de recherche systematique iterative.

## Qu'est-ce que c'est ?

Ce preset remplace les 5 commandes standard de Spec Kit par des equivalents orientes recherche, et ajoute 2 commandes supplementaires. Il permet de mener une recherche structuree sur n'importe quel sujet, en explorant des pistes de maniere iterative avec un scoring de confiance multicriteres.

**Principes :**

- Exploration iterative de pistes scorees de 0 a 100 sur 5 criteres (serieux des auteurs, popularite, applicabilite, recence, replicabilite)
- Recherche sur 3 axes : academique, industriel, public/institutionnel
- Construction automatique d'une ontologie des idees
- References BibLaTeX avec seuil de qualite configurable
- Checkpoints periodiques avec analyse de couverture des questions de recherche
- Livrable variable : article de synthese, rapport comparatif, demonstrateur, outil

**Contraintes de conception :**

- Agnostique au LLM (OpenCode, Claude Code, Cursor, Windsurf, Cline, etc.)
- Stack MCP 100% gratuite — aucune cle API payante requise
- Dependances minimales (Node.js, uv, Spec Kit)

## Pre-requis

- [Spec Kit](https://github.com/github/spec-kit) >= v0.6.0
- Node.js >= 18
- [uv](https://github.com/astral-sh/uv) (gestionnaire de packages Python)
- Un outil AI compatible Spec Kit
- [paper-search-mcp](https://github.com/your-org/paper-search-mcp) clone localement (pour la recherche et le telechargement d'articles academiques)

## Installation

```bash
# 1. Installer le preset dans votre projet
specify preset add --dev ./speckit-research-preset

# 2. Copier le fichier de contexte a la racine du projet
cp speckit-research-preset/AGENTS.md ./AGENTS.md

# 3. Copier la config MCP adaptee a votre outil AI
cp speckit-research-preset/opencode.json ./opencode.json    # OpenCode
cp speckit-research-preset/.mcp.json ./.mcp.json            # Claude Code
```

## Configuration

Avant de commencer, deux elements doivent etre personnalises dans le fichier de configuration MCP (`opencode.json` ou `.mcp.json`) :

| Element | Fichier(s) | Valeur a modifier |
|---------|-----------|-------------------|
| Chemin vers paper-search-mcp | `opencode.json` et `.mcp.json` | Remplacer `/path/to/paper-search-mcp` par le chemin absolu vers votre clone local |
| Email OpenAlex | `opencode.json` et `.mcp.json` | Remplacer `your-email@example.com` par votre email (ameliore le rate limit, optionnel) |

Pour verifier que tout fonctionne, lancez votre outil AI et verifiez que les 5 serveurs MCP se connectent.

## Stack MCP

| Serveur | Package | Role |
|---------|---------|------|
| paper-search | Instance locale | Recherche + telechargement d'articles via Sci-Hub |
| openalex | `openalex-mcp` | Metriques auteurs (h-index) et citations |
| arxiv | `@fre4x/arxiv` | Preprints ArXiv |
| web-search | `@iflow-mcp/open-websearch` | Recherche web multi-moteurs (Bing, DuckDuckGo, Brave, Startpage) |
| web-fetch | `mcp-server-fetch` | Lecture du contenu d'une URL |

Optionnel : [SearXNG](https://github.com/searxng/searxng) pour un meta-moteur auto-heberge (necessite Docker). Voir le [guide d'utilisation](GUIDE-UTILISATION.md) pour la configuration.

## Commandes

| Commande | Description |
|----------|-------------|
| `/speckit.constitution` | Etablir les principes du projet (interactif, une seule fois) |
| `/speckit.specify` | Definir un sujet de recherche avec questions et criteres de succes |
| `/speckit.plan` | Explorer les pistes (boucle iterative avec scoring) |
| `/speckit.tasks` | Evaluer, comparer et trier les pistes |
| `/speckit.implement` | Produire le livrable final |
| `/speckit.research.checkpoint` | Synthese intermediaire (auto toutes les N pistes) |
| `/speckit.research.score` | Scorer ou re-evaluer une piste specifique |

## Workflow

```
constitution → specify → plan (boucle) → checkpoint (periodique) → tasks → implement
```

1. **Constitution** — Repondre aux questions interactives pour configurer les parametres (grille de scoring, seuils, axes, checkpoint). Une seule fois par projet.
2. **Specify** — Definir le sujet : problematique, questions de recherche priorisees, perimetre, criteres de succes (Given/When/Then).
3. **Plan** — Boucle principale. Chaque iteration explore une piste, collecte des donnees via MCP, cree une fiche, score, et detecte de nouvelles pistes dans les references.
4. **Checkpoint** — Synthese automatique toutes les N pistes. Analyse la couverture des questions, detecte les desequilibres d'axes, recommande de continuer, pivoter ou conclure.
5. **Tasks** — Tableau de bord, comparaisons, filtrage, analyse de gaps.
6. **Implement** — Production du livrable (article, rapport, demo, outil).

## Structure du preset

```
speckit-research-preset/
  preset.yml               # Manifeste Spec Kit
  README.md                # Ce document
  GUIDE-UTILISATION.md     # Guide d'utilisation detaille
  opencode.json            # Config MCP pour OpenCode (a personnaliser)
  .mcp.json                # Config MCP pour Claude Code (a personnaliser)
  AGENTS.md                # Contexte projet (a copier a la racine)
  commands/                # 7 commandes Spec Kit
  templates/               # 4 templates (constitution, spec, plan, tasks)
  agents/                  # 2 agents (lead-scorer, ontologist)
```

## Compatibilite

Les commandes, templates et agents sont de simples fichiers Markdown : ils fonctionnent avec tout LLM capable de lire et suivre des instructions structurees. Seul le fichier de configuration MCP differe selon l'outil.

| Outil | Config MCP |
|-------|-----------|
| OpenCode | `opencode.json` |
| Claude Code | `.mcp.json` |
| Cursor | `.cursor/mcp.json` |
| Windsurf | Configuration MCP native |
| Cline | `.cline/mcp.json` |

## Documentation

Pour le guide d'utilisation complet (pas a pas, schema leads.json, detail des serveurs MCP, personnalisation de la grille de scoring) : [GUIDE-UTILISATION.md](GUIDE-UTILISATION.md)

## Licence

MIT
