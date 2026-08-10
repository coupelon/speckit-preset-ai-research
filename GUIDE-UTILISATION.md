# Guide d'utilisation detaille

Ce document complete le [README](README.md) avec les instructions pas a pas, les schemas de donnees, et les details techniques des serveurs MCP.

## Checklist de demarrage rapide

```
[ ] Spec Kit >= 0.14.0 installe          (specify --version)
[ ] Node.js >= 18 installe
[ ] uv installe
[ ] Projet Spec Kit initialise           (specify init --here --integration <votre outil>)
[ ] specify extension add --dev <repo>/extension     <- EN PREMIER
[ ] specify preset add --dev <repo>/preset
[ ] cp <repo>/AGENTS.md a la racine du projet
[ ] Copier la config MCP a la racine du projet (voir README > Configuration)
[ ] Personnaliser chemin paper-search + email OpenAlex dans la config MCP
[ ] Lancer votre outil AI et verifier que les 5 MCP se connectent
[ ] Verifier que /speckit.research.explore est proposee
[ ] /speckit.constitution
```

## Notes d'installation locale

C'est la seule voie operationnelle aujourd'hui — voir [Bundle](#bundle) plus bas pour pourquoi
`specify bundle install` n'est pas utilisable.

### Ordre des deux composants

**L'ordre n'est pas contraignant** — les deux sens fonctionnent, verifie sur des projets neufs.
Installer l'extension en premier reste recommande, pour une seule raison : les stubs
`/speckit.plan` et `/speckit.tasks` du preset renvoient vers les commandes `speckit.research.*`,
donc commencer par l'extension evite une fenetre pendant laquelle la redirection ne mene nulle
part. Si vous avez commence par le preset, il n'y a rien a rejouer.

Le filtre a trois segments decrit ci-dessous ne cree **pas** de contrainte d'ordre ici : il ne
s'applique qu'aux commandes `speckit.<ext-id>.<cmd>` declarees par un *preset*, et ce preset n'en
declare plus aucune. Les 4 commandes `research` sont declarees par l'extension, qui cree
elle-meme le repertoire qui les rend valides.

### Verifier l'installation

```bash
specify extension list                    # research v1.0.0, enabled
specify preset list                       # ai-research v1.0.0, priorite 10
specify preset resolve spec-template      # doit designer la couche ai-research
specify preset resolve constitution-template
```

Cote outil AI, les 4 commandes `/speckit.research.*` doivent apparaitre, et `/speckit.plan`
doit afficher son message de redirection au lieu du workflow de planification logicielle.
Pour Claude Code, les skills correspondantes sont dans `.claude/skills/speckit-research-*/`.

### Pourquoi deux composants

Spec Kit filtre les commandes a trois segments (`speckit.<ext-id>.<cmd>`) : elles ne sont
enregistrees que si `.specify/extensions/<ext-id>/` existe. Une commande `speckit.research.*`
declaree par un preset seul est donc **silencieusement ignoree** — l'assistant se plaint alors
d'une skill introuvable. Le namespace `research` appartient a l'extension ; c'est elle qui cree
le repertoire qui rend ces commandes valides.

### Le flag `--dev`

Il est necessaire pour installer depuis un chemin local. Sans lui, Spec Kit cherche dans son
catalogue en ligne et retourne une erreur "not found in catalog". Attention, la syntaxe differe
entre les deux primitives :

```bash
specify extension add --dev ./extension    # --dev est un drapeau, le chemin est l'argument
specify preset add --dev ./preset          # --dev prend le chemin en valeur
```

Vous pouvez definir une priorite si vous empilez plusieurs presets :

```bash
specify preset add --dev ./preset --priority 5
```

## Bundle

> [!WARNING]
> **`specify bundle install ai-research` ne fonctionne pas : ce bundle n'est pas publie.**
> Utilisez l'installation locale decrite ci-dessus.

`bundle.yml` epingle les deux composants (extension `research` v1.0.0 + preset `ai-research`
v1.0.0) en une unite versionnee. Mais `bundle install` resout ses composants **par identifiant
dans un catalogue** : il delegue a `specify extension add research` et
`specify preset add ai-research`. Deux consequences :

- le champ `source` d'un composant n'est pas exploite a l'installation ;
- il n'existe pas d'equivalent `--dev` pour un bundle.

Autrement dit, un bundle ne devient installable qu'une fois **ses composants** publies dans un
catalogue — publier le bundle seul ne suffirait pas. C'est un manifeste de composition et de
distribution, pas un conteneur de code.

Ce qui fonctionne des maintenant, en local :

```bash
specify bundle validate --offline        # verifie que le manifeste est bien forme et que les
                                         # references resolvent contre l'installe / l'embarque
specify bundle build --output dist/      # produit ai-research-1.0.0.zip (25 fichiers)
```

Note sur `bundle build` : le packager n'exclut que `.git`, `__pycache__` et `.DS_Store`, sans
mecanisme de type `.bundleignore`. Tout autre fichier present a la racine du depot se retrouve
dans l'archive — verifier son contenu avant publication.

## Fonctionnement des serveurs MCP

Les serveurs MCP sont lances automatiquement par votre outil AI au moment ou il en a besoin. Aucune installation prealable n'est requise pour la plupart :

- **npx** (openalex, arxiv, web-search, searxng) : `npx -y package-name` telecharge et execute le package npm automatiquement a la premiere utilisation, puis le met en cache. Seul pre-requis : Node.js >= 18.
- **uvx** (web-fetch) : equivalent de npx pour l'ecosysteme Python. Execute `mcp-server-fetch` directement, avec `--with 'mcp<2'` qui epingle le SDK `mcp` sur la serie 1.x (sans cette contrainte, le serveur ne demarre pas). Pre-requis : `uv` installe (`pip install uv` ou `brew install uv`).
- **uv run** (paper-search) : execute votre instance locale. Le chemin du repertoire est configure dans le fichier de config MCP.

Dans les trois cas, le premier lancement telecharge les dependances automatiquement, puis les utilise depuis le cache.

### (Optionnel) SearXNG — meta-moteur auto-heberge

SearXNG agrege les resultats de plusieurs moteurs (Google, Bing, DuckDuckGo, Brave, Qwant, etc.) en une seule requete. Il offre une couverture plus large, un controle total sur les moteurs actives par categorie, et aucun rate limiting externe.

```bash
docker run -d -p 8080:8080 searxng/searxng
```

Puis ajouter dans votre fichier de config MCP :

```json
"searxng": {
  "command": "npx",
  "args": ["-y", "searxng-mcp"],
  "env": { "SEARXNG_URL": "http://localhost:8080" }
}
```

SearXNG est recommande pour une exploration intensive (dizaines de requetes par session). open-websearch suffit pour un usage ponctuel grace a son acces multi-moteurs.

### Packages MCP testes et valides

Les packages ont ete testes avec un handshake JSON-RPC reel. Certains packages npm de la communaute sont casses ou incompatibles stdio :

| Package | Statut | Probleme |
|---------|--------|----------|
| `arxiv-mcp-server` | Casse | Wrapper Python, crash au demarrage |
| `@cyanheads/arxiv-mcp-server` | Casse | Dependance manquante (@opentelemetry/api) |
| `@gkzhb/crawl4ai-mcp` | Incompatible | Mode HTTP (port 8585), pas stdio |
| `crawl4ai-mcp-server` | Inexistant | Pas sur npm |
| `@iflow-mcp/mcp-crawl4ai-ts` | Incompatible | Necessite CRAWL4AI_BASE_URL externe |

## Utilisation pas a pas

### Etape 1 — Etablir la constitution (une seule fois)

```
/speckit.constitution
```

La commande demarre par une phase de decouverte interactive :

1. **Nom et description du projet** — pour contextualiser toute la recherche
2. **Objectif** — etat de l'art, benchmark, exploration de solutions, veille...
3. **Domaine** — IA, sante, geospatial, finance, etc.
4. **Orientation** — plutot academique, industrielle, ou equilibree
5. **Contraintes** — langues, periode temporelle, nombre de pistes max, etc.

A partir de vos reponses, la commande propose des valeurs adaptees pour chaque section (seuils, poids de scoring, repartition des axes, intervalle de checkpoint). Vous validez ou ajustez chaque section une par une.

Valeurs par defaut si aucune preference :

- Seuil de confiance : 60/100
- Grille : auteurs /20, popularite /20, applicabilite /25, recence /15, replicabilite /20
- Axes : academique 40-60%, industriel 20-40%, public 10-30%
- Checkpoint toutes les 10 pistes
- Langues : francais, anglais

Le resultat est sauvegarde dans `.specify/memory/constitution.md`.

### Etape 2 — Definir un sujet de recherche

```
/speckit.specify "Impact des agents LLM sur l'automatisation des workflows"
```

Cree le repertoire `specs/001-impact-agents-llm/` avec :

- `spec.md` : probleme, questions de recherche (P1/P2/P3), perimetre, criteres de succes
- `leads.json` : base de pistes vide initialisee
- `ontology.md` : squelette de l'ontologie
- `references.bib` : fichier BibLaTeX vide
- Repertoires `leads/`, `sources/academic/`, `sources/industry/`, `sources/public/`, `synthesis/`

### Etape 3 — Explorer les pistes

```
/speckit.research.explore           # Prochaine piste pending
/speckit.research.explore 5         # Explorer les 5 prochaines
/speckit.research.explore L0042     # Explorer une piste specifique
```

Pour chaque piste : recherche via MCP, collecte du contenu, creation de la fiche, scoring multicriteres, mise a jour de leads.json/ontology.md/references.bib, verification du checkpoint.

**Contenu des sources et articles non telecharges.** Le contenu complet d'un article academique
n'est pas toujours accessible (paywall, absence de Sci-Hub, preprint uniquement) : c'est normal.
Chaque piste enregistre donc un `source_status` (`full_text`, `abstract_only`, `metadata_only`,
`failed`). A la fin de `/speckit.research.explore`, un **bilan contenu** indique combien de pistes ont ete
analysees sur leur texte integral, et liste les articles restant a (re)telecharger avec leur ID,
leur statut et leur DOI/URL. Pour reessayer une piste : `/speckit.research.explore LXXXX`. Vous pouvez aussi
deposer manuellement un PDF dans `sources/[axe]/` puis re-scorer via `/speckit.research.score LXXXX`.
Le checkpoint reprend ce bilan pour eviter de conclure une recherche sur des sources superficielles.

### Etape 4 — Checkpoint (automatique toutes les N pistes)

```
/speckit.research.checkpoint
```

Se declenche automatiquement ou peut etre appele manuellement. Produit : metriques globales, couverture des questions (BONNE / PARTIELLE / INSUFFISANTE / NON COUVERTE), alertes, recommandation.

### Etape 5 — Evaluer et comparer

```
/speckit.research.dashboard                    # Tableau de bord complet
/speckit.research.dashboard L0001 L0003 L0007  # Comparaison de pistes
/speckit.research.dashboard top 5              # Les 5 meilleures
/speckit.research.dashboard gaps               # Lacunes identifiees
/speckit.research.dashboard axe:industriel     # Filtrer par axe
```

### Etape 6 — Produire le livrable

```
/speckit.implement
```

Genere le livrable selon le type defini dans spec.md : article de synthese, rapport comparatif, demonstrateur ou outil.

### Commande utilitaire : re-scorer

```
/speckit.research.score L0042     # Re-scorer une piste
/speckit.research.score all       # Re-scorer toutes les pistes
```

## Routage des outils MCP

Les commandes de l'extension routent automatiquement vers le bon outil, mais voici la logique explicite :

- **Trouver un article academique** → `paper-search` (pas `web-fetch`, pas `web-search`)
- **Lire le contenu d'un article** → `paper-search` telecharge via Sci-Hub (pas `web-fetch`)
- **Connaitre le h-index d'un auteur** → `openalex`
- **Chercher un preprint ArXiv** → `arxiv` (recherche par mot-cle, auteur ou categorie)
- **Trouver un blog technique / produit / rapport** → `web-search` puis `web-fetch` pour lire la page
- **Lire une documentation multi-pages** → `web-fetch` appele sur chaque page en suivant les liens

## Schema leads.json

```json
{
  "metadata": {
    "research_id": "001",
    "topic": "impact-agents-llm",
    "created_at": "2026-04-13T10:00:00Z",
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
  "leads": [
    {
      "id": "L0001",
      "title": "Titre de la piste",
      "url": "https://...",
      "axis": "academic",
      "status": "explored",
      "added_at": "2026-04-13T10:00:00Z",
      "explored_at": "2026-04-13T11:00:00Z",
      "source_lead": null,
      "source_status": "full_text",
      "content_incomplete": false,
      "questions": ["Q1", "Q2"],
      "score": {
        "total": 78,
        "details": {
          "authors": { "value": 16, "max": 20, "justification": "..." },
          "popularity": { "value": 12, "max": 20, "justification": "..." },
          "applicability": { "value": 20, "max": 25, "justification": "..." },
          "recency": { "value": 15, "max": 15, "justification": "..." },
          "replicability": { "value": 15, "max": 20, "justification": "..." }
        },
        "scored_at": "2026-04-13T11:00:00Z"
      }
    }
  ]
}
```

## Personnalisation

### Grille de scoring

Editer `.specify/memory/constitution.md` apres avoir lance `/speckit.constitution` :

- Changer les poids (ex: applicabilite a /30 au lieu de /25)
- Modifier les baremes (ex: h-index >= 50 pour 20/20)
- Ajuster le seuil de confiance (ex: 70 au lieu de 60)

### Intervalle de checkpoint

Dans `leads.json`, champ `metadata.checkpoint_interval` (defaut: 10).

### Ajouter un axe de recherche

Ajouter une entree dans `statistics.by_axis` de leads.json et creer le repertoire `sources/[nouvel-axe]/`.

### Ajouter un serveur MCP

Ajouter la configuration dans votre fichier de config MCP (`opencode.json` ou `.mcp.json`) et mettre a jour les commandes qui l'utilisent.
