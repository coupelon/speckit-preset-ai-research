# AI Research — Bundle Spec Kit

Bundle pour [GitHub Spec Kit](https://github.com/github/spec-kit) qui transforme le workflow spec-driven en un framework de recherche systematique iterative.

## Qu'est-ce que c'est ?

Le bundle `ai-research` compose deux primitives Spec Kit, chacune dans son role :

| Composant | Role | Contenu |
|-----------|------|---------|
| **Extension `research`** | Apporter une activite que le cycle core ne connait pas | 4 commandes `speckit.research.*`, 2 agents, 2 templates, la config MCP |
| **Preset `ai-research`** | Relire les creneaux core en vocabulaire recherche | 3 commandes core surchargees, 2 stubs de redirection, 2 templates |

Ce decoupage suit la doctrine Spec Kit : un preset *« customize how Spec Kit works — overriding command files, template files and script files without changing any tooling »*, une extension *« add new capabilities — domain-specific commands, external tool integrations, quality gates »*. L'exploration de pistes n'occupe aucun creneau du cycle core : elle vit donc dans l'extension, pas dans un `speckit.plan` detourne.

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

- [Spec Kit](https://github.com/github/spec-kit) >= v0.14.0 (les bundles et la resolution des commandes d'extension necessitent cette version)
- Node.js >= 18
- [uv](https://github.com/astral-sh/uv) (gestionnaire de packages Python)
- Un outil AI compatible Spec Kit
- [paper-search-mcp](https://github.com/your-org/paper-search-mcp) clone localement (pour la recherche et le telechargement d'articles academiques)

## Installation

### En developpement local (depuis ce depot)

L'extension doit etre installee **avant** le preset : les stubs de redirection du preset
referencent les commandes `speckit.research.*`.

```bash
# 1. L'extension (l'activite de recherche)
specify extension add --dev ./extension

# 2. Le preset (la localisation des creneaux core)
specify preset add --dev ./preset

# 3. Copier le fichier de contexte a la racine du projet
cp AGENTS.md ./AGENTS.md

# 4. Copier la config MCP adaptee a votre outil AI (UNE seule des deux lignes)

# -> Si vous utilisez Claude Code :
cp extension/.mcp.json ./.mcp.json

# -> Si vous utilisez OpenCode :
cp extension/opencode.json ./opencode.json
```

### Depuis un catalogue (une fois le bundle publie)

```bash
specify bundle install ai-research
```

`specify bundle install` resout les deux composants epingles dans `bundle.yml` et les installe
via la machinerie de chaque primitive. Cette voie exige que l'extension et le preset soient
publies dans un catalogue : en local, utilisez les deux commandes `--dev` ci-dessus.

Pour produire l'archive distribuable : `specify bundle build --output dist/`.
Pour verifier le manifeste : `specify bundle validate --offline`.

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

Creneaux du cycle core, relus en vocabulaire recherche (**preset**) :

| Commande | Description |
|----------|-------------|
| `/speckit.constitution` | Etablir les principes du projet (interactif, une seule fois) |
| `/speckit.specify` | Definir un sujet de recherche avec questions et criteres de succes |
| `/speckit.implement` | Produire le livrable final |

Activite de recherche (**extension**) :

| Commande | Description |
|----------|-------------|
| `/speckit.research.explore` | Explorer les pistes (boucle iterative avec scoring) |
| `/speckit.research.checkpoint` | Synthese intermediaire (auto toutes les N pistes) |
| `/speckit.research.dashboard` | Evaluer, comparer et trier les pistes |
| `/speckit.research.score` | Scorer ou re-evaluer une piste specifique |

Creneaux non utilises : `/speckit.plan` et `/speckit.tasks` n'ont pas d'equivalent dans cette
methodologie. Le preset les remplace par des stubs qui redirigent vers
`/speckit.research.explore` et `/speckit.research.dashboard`, pour eviter qu'un reflexe de
frappe ne fasse basculer l'utilisateur dans le workflow de developpement logiciel.

## Workflow

```
constitution → specify → research.explore (boucle) → research.checkpoint (periodique) → research.dashboard → implement
```

1. **Constitution** — Repondre aux questions interactives pour configurer les parametres (grille de scoring, seuils, axes, checkpoint). Une seule fois par projet.
2. **Specify** — Definir le sujet : problematique, questions de recherche priorisees, perimetre, criteres de succes (Given/When/Then).
3. **Research.explore** — Boucle principale. Chaque iteration explore une piste, collecte des donnees via MCP, cree une fiche, score, et detecte de nouvelles pistes dans les references.
4. **Research.checkpoint** — Synthese automatique toutes les N pistes. Analyse la couverture des questions, detecte les desequilibres d'axes, recommande de continuer, pivoter ou conclure.
5. **Research.dashboard** — Tableau de bord, comparaisons, filtrage, analyse de gaps.
6. **Implement** — Production du livrable (article, rapport, demo, outil).

## Structure du depot

```
speckit-preset-ai-research/
  bundle.yml                 # Manifeste de bundle (compose extension + preset)
  README.md                  # Ce document
  GUIDE-UTILISATION.md       # Guide d'utilisation detaille
  AGENTS.md                  # Contexte projet (a copier a la racine)

  extension/                 # Extension `research` — l'activite de recherche
    extension.yml
    .mcp.json                # Config MCP pour Claude Code (a personnaliser)
    opencode.json            # Config MCP pour OpenCode (a personnaliser)
    commands/                # explore, checkpoint, score, dashboard
    templates/               # lead-template, dashboard-template
    agents/                  # lead-scorer, ontologist

  preset/                    # Preset `ai-research` — localisation du cycle core
    preset.yml
    commands/                # constitution, specify, implement + stubs plan/tasks
    templates/               # constitution-template, spec-template
```

Une fois installee, l'extension est deployee dans `.specify/extensions/research/` : les commandes
y referencent leurs templates et agents par chemin (ex.
`.specify/extensions/research/templates/lead-template.md`).

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
