---
description: "Synthese intermediaire et validation periodique (toutes les 10 pistes)"
---

## User Input
$ARGUMENTS

## Contexte

Le checkpoint est un arret obligatoire dans l'exploration pour :
- Faire le point sur la couverture des questions de recherche
- Valider la qualite des pistes explorees
- Decider de continuer, pivoter ou conclure
- Eviter l'exploration sans fin

Il se declenche automatiquement toutes les N pistes (defaut: 10, configurable dans `leads.json`).

## Pre-requis

CRITICAL : Verifier que les fichiers suivants existent dans le repertoire du sujet courant (`specs/###-slug/`) :
- `spec.md`
- `leads.json`
- `ontology.md`
- `references.bib`
- `.specify/memory/constitution.md`

## Workflow

### Etape 1 — Collecte des metriques

Lire `leads.json` et calculer :

```
Checkpoint #N — [date ISO]

Pistes totales : X
  - explored : X (score moyen : XX)
  - pending : X
  - abandoned : X

Par axe :
  - academique : X explored (moy: XX) | Y pending
  - industriel : X explored (moy: XX) | Y pending
  - public : X explored (moy: XX) | Y pending

References.bib : X entrees (seuil >= 60)

Contenu des sources :
  - full_text     : X
  - abstract_only : X
  - metadata_only : X
  - failed        : X
```

Si des pistes ont `"content_incomplete": true` dans `leads.json`, lister les articles dont le
contenu n'a pas ete atteint, pour permettre de les retelecharger avant de conclure :

```
Articles a (re)telecharger (contenu non atteint) :
  | ID    | Statut        | Titre           | URL / DOI    |
  |-------|---------------|-----------------|--------------|
  | LXXXX | abstract_only | [titre complet] | [DOI ou URL] |
  | LXXXX | failed        | [titre complet] | [DOI ou URL] |
```

### Etape 2 — Couverture des questions

Pour chaque question de recherche dans spec.md :

```
Q1 [Enonce court] — Priorite P1
  Pistes : L0001 (85), L0003 (72), L0008 (68)
  Couverture : BONNE (3 sources, score moyen 75)

Q2 [Enonce court] — Priorite P2
  Pistes : L0005 (55)
  Couverture : INSUFFISANTE (1 source, score < seuil)

Q3 [Enonce court] — Priorite P3
  Pistes : aucune
  Couverture : NON COUVERTE
```

Niveaux de couverture :
- **BONNE** : >= 2 pistes avec score >= seuil
- **PARTIELLE** : 1 piste avec score >= seuil, ou >= 2 pistes sous le seuil
- **INSUFFISANTE** : 1 seule piste sous le seuil
- **NON COUVERTE** : aucune piste

### Etape 3 — Analyse de l'ontologie

Lire `ontology.md` et evaluer :
- Nombre de concepts identifies
- Profondeur de l'arbre (niveaux de hierarchie)
- Branches isolees (concepts sans lien)
- Clusters denses (concepts tres lies)

### Etape 4 — Alertes

Generer des alertes si :
- Un axe a 0 piste exploree (alerte : **desequilibre**)
- Le score moyen global < seuil constitution (alerte : **qualite**)
- Une question P1 est NON COUVERTE (alerte : **critique**)
- Le ratio pending/explored > 3 (alerte : **explosion des pistes**)
- Aucune nouvelle piste generee sur les 5 dernieres explorations (alerte : **saturation**)
- Plus de 30% des pistes explorees n'ont pas de contenu complet (`source_status` != `full_text`) (alerte : **contenu superficiel**)

### Etape 5 — Decision

Proposer une des actions suivantes :

| Situation | Action recommandee |
|-----------|-------------------|
| Questions P1 bien couvertes, P2 partielles | Continuer `/speckit.plan` ciblant P2 |
| Toutes les questions couvertes | Passer a `/speckit.implement` |
| Qualite insuffisante | Relancer `/speckit.research.score` sur les pistes douteuses |
| Desequilibre d'axe | Cibler l'axe sous-represente dans `/speckit.plan` |
| Explosion de pistes | Augmenter le seuil et abandonner les pistes < nouveau seuil |
| Saturation | Passer a `/speckit.implement` ou elargir le perimetre |

### Etape 6 — Sauvegarde

1. Ecrire `synthesis/checkpoint-N.md` avec le rapport complet
2. Mettre a jour `leads.json` :
   - `metadata.last_checkpoint` → date ISO
   - `metadata.checkpoint_number` → N
   - `metadata.next_checkpoint_at` → total_explored + intervalle
3. Si l'utilisateur valide des ajustements :
   - Modifier le `confidence_threshold` si demande
   - Modifier le `checkpoint_interval` si demande
   - Abandonner les pistes designees (`"abandoned"`)

## Sortie

```
=== Checkpoint #N — [Titre du sujet] ===

[Metriques]
[Couverture des questions]
[Alertes]

Recommandation : [action]
Prochain checkpoint dans : X pistes

Valider ? Ajustements possibles :
  - Modifier le seuil de confiance
  - Modifier l'intervalle de checkpoint
  - Abandonner des pistes specifiques
  - Continuer tel quel
```

## Prochaine etape

- Continuer → `/speckit.plan`
- Produire → `/speckit.implement`
- Evaluer → `/speckit.tasks`
