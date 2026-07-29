---
description: "Scorer ou re-evaluer une piste avec metriques MCP"
---

## User Input
$ARGUMENTS

## Contexte

Le scoring est un processus reproductible et trace qui attribue un score de confiance (0-100)
a une piste en utilisant les outils MCP pour collecter des metriques objectives.

Cette commande peut etre utilisee :
- Lors de l'exploration initiale (appelee par `/speckit.research.explore`)
- Pour re-evaluer une piste apres un certain temps
- Pour scorer une piste ajoutee manuellement

## Pre-requis

CRITICAL :
- `leads.json` doit exister avec la piste ciblee
- `.specify/memory/constitution.md` doit exister (grille de scoring)
- $ARGUMENTS doit contenir un ID de piste (ex: "L0042") ou "all" pour tout re-scorer

## Grille de scoring

| Critere | Poids | Source MCP | Methode |
|---------|-------|-----------|---------|
| Serieux des auteurs | /20 | openalex | h-index moyen des auteurs |
| Popularite de la source | /20 | openalex | Nombre de citations |
| Applicabilite | /25 | Analyse du contenu | Presence de demo/code/produit |
| Recence | /15 | Date de publication | Penalite par annee d'anciennete |
| Replicabilite | /20 | Analyse du contenu | Code/methodo/donnees disponibles |

CRITICAL : Les criteres **Applicabilite** et **Replicabilite** exigent le contenu reel de la source.
Si la piste a `source_status` != `full_text` (contenu non telecharge), ces deux criteres ne peuvent
pas etre evalues de maniere fiable : les plafonner et le signaler dans la justification
(ex: "evalue sur resume seul — texte integral non disponible"). Si l'utilisateur a depose
manuellement un PDF dans `sources/[axe]/`, le lire et mettre a jour `source_status` a `full_text`
(et `content_incomplete` a `false`) lors du re-scoring.

## Workflow

### Etape 1 — Identification

- Si $ARGUMENTS = ID unique (ex: "L0042") → scorer cette piste
- Si $ARGUMENTS = liste d'IDs (ex: "L0001 L0003") → scorer chaque piste
- Si $ARGUMENTS = "all" → re-scorer toutes les pistes `"explored"`
- Si $ARGUMENTS = "pending" → scorer les pistes `"pending"` qui ont deja du contenu

### Etape 2 — Collecte des metriques via MCP

Pour chaque piste a scorer :

**2a. Serieux des auteurs (/20)**
```
→ openalex : chercher chaque auteur
→ Recuperer h-index de chaque auteur
→ Calculer h-index moyen
→ Bareme :
   h >= 40 → 20/20
   h >= 25 → 16/20
   h >= 15 → 12/20
   h >= 8  → 8/20
   h >= 3  → 4/20
   h < 3 ou inconnu → 2/20
   Source non-academique sans auteur identifie → 5/20 (neutre)
```

**2b. Popularite (/20)**
```
→ openalex : nombre de citations de la source
→ Bareme :
   citations >= 500 → 20/20
   citations >= 200 → 16/20
   citations >= 50  → 12/20
   citations >= 10  → 8/20
   citations >= 1   → 4/20
   0 citations      → 2/20
   Source non-academique → estimer par presence web/trafic → 5-15/20
```

**2c. Applicabilite (/25)**
```
→ Analyser le contenu de la fiche leads/LXXXX-slug.md
→ Bareme :
   Produit commercial actif        → 25/25
   Demonstrateur fonctionnel/code  → 20/25
   Prototype documente             → 15/25
   Methodologie detaillee          → 10/25
   Cadre theorique uniquement      → 5/25
```

**2d. Recence (/15)**
```
→ Date de publication
→ Annee courante = 2026
→ Bareme :
   0-1 an  (2025-2026) → 15/15
   2-3 ans (2023-2024) → 12/15
   4-5 ans (2021-2022) → 8/15
   6-8 ans (2018-2020) → 4/15
   > 8 ans             → 2/15
```

**2e. Replicabilite (/20)**
```
→ Analyser la disponibilite :
→ Bareme :
   Code open-source + donnees + methodo  → 20/20
   Code open-source + methodo            → 16/20
   Methodo detaillee reproductible       → 12/20
   Methodo partielle                     → 8/20
   Aucune methodo/donnees/code           → 4/20
```

### Etape 3 — Calcul et justification

Pour chaque piste :
1. Calculer le score total = somme des 5 criteres
2. Generer une justification textuelle pour chaque critere
3. Comparer avec le score precedent si re-evaluation

### Etape 4 — Mise a jour

1. `leads.json` : mettre a jour le score, les sous-scores, la date d'evaluation et, si le contenu
   a ete obtenu depuis, `source_status` / `content_incomplete`
2. `leads/LXXXX-slug.md` : mettre a jour la section scoring
3. Si le score passe au-dessus du seuil → ajouter a `references.bib` si absent
4. Si le score passe en-dessous du seuil → signaler (ne pas supprimer de references.bib)

### Format de score dans leads.json

```json
{
  "id": "L0042",
  "score": {
    "total": 78,
    "details": {
      "authors": { "value": 16, "max": 20, "justification": "h-index moyen 28" },
      "popularity": { "value": 12, "max": 20, "justification": "87 citations" },
      "applicability": { "value": 20, "max": 25, "justification": "Demo GitHub fonctionnelle" },
      "recency": { "value": 15, "max": 15, "justification": "Publie en 2025" },
      "replicability": { "value": 15, "max": 20, "justification": "Code disponible, pas de donnees" }
    },
    "scored_at": "2026-04-13T10:00:00Z",
    "previous_score": null
  }
}
```

## Sortie

```
Piste L0042 : [Titre]
  Serieux auteurs :   16/20 (h-index moyen: 28)
  Popularite :        12/20 (87 citations)
  Applicabilite :     20/25 (demo GitHub)
  Recence :           15/15 (2025)
  Replicabilite :     15/20 (code sans donnees)
  ─────────────────────────
  TOTAL :             78/100 [>= seuil 60 ✓]

  Changement : [nouveau | +5 | -3 | stable]
  References.bib : [ajoute | deja present | sous le seuil]
```

## Prochaine etape

- Continuer l'exploration → `/speckit.research.explore`
- Voir le classement → `/speckit.research.dashboard`
- Checkpoint → `/speckit.research.checkpoint`
