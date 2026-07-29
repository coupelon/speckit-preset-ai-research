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

Deux voies existent. **Seule l'installation locale fonctionne aujourd'hui** — voir l'avertissement
de la section bundle.

### Voie 1 — Installation locale, composant par composant (la seule operationnelle)

Depuis la racine d'un projet Spec Kit deja initialise (`specify init`), en supposant ce depot
clone dans `../speckit-preset-ai-research` :

```bash
REPO=../speckit-preset-ai-research

# 1. L'extension `research` — l'activite de recherche.
#    Recommande en premier : les stubs du preset renvoient vers speckit.research.*,
#    donc installer l'extension d'abord evite une fenetre ou la redirection ne mene
#    nulle part. L'ordre inverse fonctionne aussi (voir le guide).
specify extension add --dev "$REPO/extension"

# 2. Le preset `ai-research` — la localisation des creneaux core.
specify preset add --dev "$REPO/preset"

# 3. Le fichier de contexte, a la racine du projet.
cp "$REPO/AGENTS.md" ./AGENTS.md

# 4. La config MCP adaptee a votre outil AI (UNE seule des deux lignes).
cp "$REPO/extension/.mcp.json"     ./.mcp.json       # Claude Code
cp "$REPO/extension/opencode.json" ./opencode.json   # OpenCode
```

Attention, la syntaxe de `--dev` differe entre les deux primitives : c'est un **drapeau** pour
`extension add` (le chemin est l'argument positionnel) et une **option a valeur** pour
`preset add`. Les deux formes ci-dessus sont correctes telles quelles.

Verifier le resultat :

```bash
specify extension list                  # research v1.0.0
specify preset list                     # ai-research v1.0.0 (priorite 10)
specify preset resolve spec-template    # doit designer la couche ai-research
```

Vous devez voir apparaitre les 4 commandes `/speckit.research.*` dans votre outil AI, et
`/speckit.plan` / `/speckit.tasks` doivent afficher leur message de redirection. Si les commandes
`research` sont absentes, c'est l'etape 1 qui a echoue : verifier avec `specify extension list`
que `research` est bien installee et `enabled`.

Etape suivante : [Configuration](#configuration) — deux valeurs a personnaliser dans le fichier MCP.

### Voie 2 — Installation par bundle

> [!WARNING]
> **Indisponible : ce bundle n'est pas publie.** `specify bundle install` resout ses composants
> **par identifiant dans un catalogue** — l'installation delegue a `specify extension add research`
> et `specify preset add ai-research` (voir `bundler/services/primitives.py`). Le champ `source`
> d'un composant n'est pas exploite a l'installation, et il n'existe pas d'equivalent `--dev`
> pour un bundle : **il n'est donc pas possible d'installer ce depot par `bundle install`
> aujourd'hui.** Utilisez la voie 1.

Une fois l'extension et le preset publies dans un catalogue, l'installation deviendra :

```bash
specify bundle install ai-research
```

En attendant, le manifeste `bundle.yml` sert deja a deux choses, qui fonctionnent en local :

```bash
specify bundle validate --offline      # verifier que le manifeste est bien forme
specify bundle build --output dist/    # produire l'archive versionnee ai-research-1.0.0.zip
```

`bundle validate --offline` verifie les references contre les composants installes ou embarques,
sans acces reseau. `bundle build` produit l'artefact distribuable qui servira a la publication.

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
