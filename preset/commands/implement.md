---
description: "Produire un livrable a partir des pistes selectionnees"
---

## User Input
$ARGUMENTS

## Contexte

L'"Implement" dans le contexte recherche correspond a la production du livrable final.
Contrairement au developpement logiciel classique, le livrable peut etre tres variable :
article de synthese, etat de l'art, demonstrateur, outil, rapport, comparatif.

## Pre-requis

CRITICAL : Verifier que les fichiers suivants existent dans le repertoire du sujet courant (`specs/###-slug/`) :
- `spec.md` (sujet defini avec livrables attendus)
- `leads.json` (pistes explorees et scorees)
- `ontology.md` (idees structurees)
- `references.bib` (sources validees)
- Au moins 1 checkpoint valide (fichier `synthesis/checkpoint-N.md`)

## Workflow

### Etape 1 — Preparation

1. Lire `spec.md` section "Livrables attendus" pour determiner le type de sortie
2. Lire le dernier checkpoint pour l'etat de la recherche
3. Charger `leads.json` : filtrer les pistes `"explored"` avec score >= seuil constitution
4. Charger `ontology.md` pour la structure des idees
5. Charger `references.bib` pour les citations

### Etape 2 — Selection du mode

Selon le type de livrable defini dans spec.md :

#### Mode A : Article de synthese / Etat de l'art

1. Structurer selon les axes de l'ontologie
2. Pour chaque section :
   - Synthetiser les pistes pertinentes (minimum 2 sources par affirmation)
   - Citer les references au format [auteur, annee]
   - Identifier les consensus et les controverses
3. Inclure :
   - Introduction avec problematique et questions de recherche
   - Corps structure par themes (pas par pistes)
   - Tableau comparatif des approches
   - Discussion : limites, biais, gaps restants
   - Conclusion avec recommandations
4. Generer la bibliographie a partir de `references.bib`
5. Ecrire dans `synthesis/article-final.md`

#### Mode B : Rapport comparatif

1. Selectionner les pistes par le critere donne (top N, axe, question)
2. Pour chaque piste :
   - Resume de l'approche
   - Forces et faiblesses
   - Applicabilite au contexte
3. Matrice de comparaison multicriteres
4. Recommandation argumentee
5. Ecrire dans `synthesis/rapport-comparatif.md`

#### Mode C : Demonstrateur / Prototype

1. Identifier la piste ou combinaison de pistes avec le meilleur score d'applicabilite
2. Documenter l'architecture proposee basee sur les sources
3. Lister les composants, dependances, outils necessaires
4. Creer un plan d'implementation step-by-step
5. Si du code source est disponible dans les pistes :
   - Recuperer via MCP (`web-fetch` ou `arxiv` pour le code supplementaire)
   - Adapter au contexte du projet
6. Ecrire dans `synthesis/demonstrateur/`

#### Mode D : Outil / Implementation

1. A partir des pistes les mieux scorees :
   - Extraire les specifications techniques
   - Identifier les patterns communs
   - Definir l'API ou l'interface
2. Generer la structure du projet
3. Implementer en suivant les meilleures pratiques identifiees
4. Documenter les choix avec references aux pistes sources
5. Ecrire dans `synthesis/implementation/`

### Etape 3 — Verification

Pour tout type de livrable :
1. Verifier que chaque question de recherche de spec.md est addressee
2. Verifier que les criteres de succes de spec.md sont satisfaits
3. Lister les criteres non satisfaits avec justification
4. Verifier la coherence des citations avec references.bib

### Etape 4 — Meta-donnees

Generer `synthesis/metadata.json` :
```json
{
  "research_id": "###",
  "topic": "[slug]",
  "deliverable_type": "[article|rapport|demo|outil]",
  "generated_at": "[ISO date]",
  "leads_used": ["L0001", "L0003", "L0007"],
  "leads_total_explored": N,
  "avg_confidence_score": XX,
  "questions_covered": ["Q1", "Q2", "Q3"],
  "questions_uncovered": ["Q4"],
  "success_criteria_met": X,
  "success_criteria_total": Y
}
```

## Sortie

```
=== Livrable genere — [Titre du sujet] ===

Type : [article|rapport|demo|outil]
Pistes utilisees : N (score moyen : XX/100)
Questions couvertes : X/Y
Criteres de succes : X/Y satisfaits

Fichiers generes :
  synthesis/[fichier-principal]
  synthesis/metadata.json

Criteres non satisfaits :
  - [critere] : [raison]
```

## Prochaine etape

- Criteres manquants → retour a `/speckit.research.explore` pour explorer davantage
- Livrable satisfaisant → revue manuelle et publication
- Ajustements → modifier et relancer `/speckit.implement`
