---
description: "Explorer iterativement les pistes de recherche avec scoring de confiance et outils MCP"
---

## User Input
$ARGUMENTS

## Contexte

Le "Plan" dans le contexte recherche correspond a l'exploration des pistes.
C'est le coeur du framework : une boucle iterative qui collecte, analyse, score et trace chaque piste.

## Pre-requis

CRITICAL : Verifier que les fichiers suivants existent dans le repertoire du sujet courant (`specs/###-slug/`) :
- `spec.md` (sujet defini)
- `leads.json` (initialise)

Si `leads.json` est vide (aucune piste), commencer par une recherche initiale.

## Regles d'utilisation des outils MCP

CRITICAL : Respecter ces regles de routage. Ne pas utiliser `web-fetch` quand un outil specialise existe.

### Hierarchie des outils par type de source

**Sources academiques (articles, papiers, preprints) :**
1. **`paper-search`** — TOUJOURS utiliser en premier pour chercher ET recuperer les articles academiques. Cet outil accede a 20+ bases (Semantic Scholar, CrossRef, OpenAlex, CORE, etc.) et peut telecharger le contenu complet des articles via un miroir Sci-Hub.
2. **`arxiv`** — Pour les preprints ArXiv specifiquement : recherche par mot-cle, par auteur, par categorie. Outils disponibles : `arxiv_search_papers`, `arxiv_get_paper`, `arxiv_search_by_author`, `arxiv_search_by_category`.
3. **`openalex`** — Pour les metriques auteurs (h-index, affiliations) et les citations. Ne PAS utiliser pour telecharger des articles.

**Sources industrielles (blogs techniques, produits, documentation) :**
1. **`web-search`** (ou `searxng`) — Pour trouver les URLs pertinentes.
2. **`web-fetch`** — Pour lire le contenu d'une page web apres avoir obtenu l'URL via recherche. Peut etre appele plusieurs fois pour suivre les liens d'une documentation multi-pages.

**Sources publiques (rapports institutionnels, donnees ouvertes) :**
1. **`web-search`** (ou `searxng`) — Pour trouver les URLs.
2. **`web-fetch`** — Pour lire le contenu des pages. Pour un rapport PDF, preferer `paper-search` s'il est indexe.

### Ce que `web-fetch` fait et ne fait pas
- **Fait** : recuperer le contenu d'une URL unique, le convertir en Markdown lisible
- **Ne fait pas** : rechercher des articles, telecharger des PDFs academiques, crawler un site entier
- **Quand l'utiliser** : uniquement quand on a deja une URL precise d'une page web a lire
- **Pour un site multi-pages** : appeler `web-fetch` sur chaque page individuellement en suivant les liens

### Erreurs a eviter
- NE PAS utiliser `web-fetch` sur un DOI ou URL academique → utiliser `paper-search` qui accede au contenu complet
- NE PAS utiliser `web-fetch` pour chercher des articles → utiliser `paper-search` ou `web-search`
- NE PAS utiliser `openalex` pour recuperer le contenu d'un article → c'est un outil de metriques

## Workflow

### Phase 0 : Recherche initiale (si aucune piste)

Si `leads.json` ne contient aucune piste :
1. Lire `spec.md` pour les questions de recherche et le perimetre
2. Lire `.specify/memory/constitution.md` pour les principes
3. Generer les premieres pistes :
   - **Academique** : `paper-search` avec 3-5 requetes sur les mots-cles principaux, puis `arxiv` pour les preprints recents
   - **Industriel** : `web-search` ou `searxng` avec des requetes orientees produits/outils/solutions
   - **Public** : `web-search` ou `searxng` avec des requetes orientees rapports/institutions/donnees
   - **Metriques** : `openalex` pour identifier les experts et les articles les plus cites dans le domaine
4. Ajouter chaque piste dans `leads.json` avec statut `"pending"`
5. Verifier les doublons avant chaque ajout (par titre et URL)

### Phase 1 : Exploration iterative

Pour chaque piste a explorer :

**Etape 1 — Selection**
- Si $ARGUMENTS contient un ID (ex: "L0042"), explorer cette piste
- Si $ARGUMENTS contient un nombre (ex: "5"), explorer les N prochaines
- Sinon, selectionner la prochaine piste `"pending"` par priorite :
  1. Pistes issues de sources a haut score
  2. Pistes couvrant un axe sous-represente
  3. Pistes les plus anciennes (FIFO)

**Etape 2 — Collecte via MCP**

Selon l'axe de la piste :

| Axe | Etape 1 : Recherche | Etape 2 : Contenu | Etape 3 : Metriques |
|-----|---------------------|-------------------|---------------------|
| Academique | `paper-search` | `paper-search` (telechargement) ou `arxiv` (preprint) | `openalex` (h-index, citations) |
| Industriel | `web-search` / `searxng` | `web-fetch` (page par page) | — |
| Public | `web-search` / `searxng` | `web-fetch` (page) ou `paper-search` (si rapport indexe) | — |

Pour chaque source :
- Telecharger le contenu dans `sources/[academic|industry|public]/`
- Recuperer les metriques auteurs via `openalex` si academique

**Etape 3 — Analyse et fiche**

Creer `leads/LXXXX-slug.md` selon le template plan-template.md avec :
- Resume (3-5 phrases)
- Idees cles avec chemin dans l'ontologie
- Approche proposee par les auteurs
- Apport aux criteres de succes de spec.md
- Nouvelles pistes detectees (voir ci-dessous)

**Detection de nouvelles pistes dans les references :**

CRITICAL : Ne JAMAIS copier le texte brut d'une citation (ex: "[23]", "[Smith et al., 2024]", "(voir section 3)").
Pour chaque reference interessante trouvee dans l'article :

1. **Identifier la reference complete** dans la bibliographie de l'article (section References/Bibliography)
2. **Extraire** : titre complet de l'article, auteurs, annee, DOI ou URL si disponible
3. **Si la bibliographie n'est pas accessible** : utiliser `paper-search` avec le texte de citation pour retrouver l'article (ex: rechercher "Smith 2024 anomaly detection" plutot que copier "[Smith et al., 2024]")
4. **Si impossible a resoudre** : noter dans la colonne Titre "A identifier — cite comme [texte original]" pour traitement ulterieur

Chaque nouvelle piste doit avoir au minimum un **titre complet exploitable** pour une recherche ulterieure.

**Etape 4 — Scoring**

Calculer le score de confiance (0-100) selon la grille de la constitution :

| Critere | Source de donnees MCP |
|---------|----------------------|
| Serieux auteurs (/20) | openalex : h-index |
| Popularite (/20) | openalex : citation count |
| Applicabilite (/25) | Presence de demo/produit/code |
| Recence (/15) | Date de publication |
| Replicabilite (/20) | Disponibilite code/methodo/donnees |

**Etape 5 — Mise a jour**

1. `leads.json` : piste → `"explored"`, score, nouvelles pistes `"pending"` (dedupliquees)
2. `ontology.md` : integrer les idees avec references [LXXXX]
3. `references.bib` : si score >= seuil constitution, ajouter :
```bibtex
@online{cle-unique,
  author    = {Nom, Prenom},
  title     = {Titre},
  year      = {2024},
  url       = {https://...},
  urldate   = {2026-04-13},
  file      = {sources/categorie/fichier.pdf},
  keywords  = {mot-cle1, mot-cle2},
  note      = {Score: XX/100 — Piste LXXXX}
}
```

**Etape 6 — Checkpoint**

Apres chaque piste :
- Incrementer `total_explored` dans `leads.json`
- Si `total_explored` atteint `next_checkpoint_at` → ARRETER et declencher `/speckit.research.checkpoint`
- Sinon, afficher un resume et passer a la piste suivante

## Sortie par piste

```
Piste LXXXX : [Titre]
  Outil MCP : [paper-search|arxiv|web-fetch] | Axe : [academique/industriel/public]
  Score : XX/100
  Nouvelles pistes : +N | References.bib : [Oui/Non]
  Checkpoint dans : Z pistes
```

## Prochaine etape

- Checkpoint atteint → `/speckit.research.checkpoint`
- Pistes restantes → continuer `/speckit.plan`
- Re-evaluer → `/speckit.research.score L0042`
- Plus de pistes → `/speckit.implement`
