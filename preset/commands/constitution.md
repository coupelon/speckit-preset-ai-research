---
description: "Etablir les principes et standards de qualite de la recherche"
---

## Contexte

Tu configures les principes de gouvernance d'un projet de recherche systematique.
Ces principes sont immuables pendant toute la duree du projet et guident toutes les decisions.

## Instructions

### Phase 1 — Decouverte du projet

AVANT de presenter les valeurs par defaut, poser les questions suivantes a l'utilisateur
pour comprendre le contexte et adapter les reglages :

1. **Nom du projet** : "Comment s'appelle votre projet de recherche ?"
2. **Description** : "En quelques phrases, que fait ce projet ? Quel probleme cherche-t-il a resoudre ?"
3. **Objectif de la recherche** : "Quel est l'objectif principal de cette recherche ? (etat de l'art, benchmark, exploration de solutions, veille technologique, autre)"
4. **Domaine** : "Dans quel domaine ou secteur se situe la recherche ? (ex: IA, sante, geospatial, finance, etc.)"
5. **Orientation** : "Votre recherche est plutot orientee vers :"
   - La theorie et la litterature academique
   - Les solutions industrielles et outils existants
   - Les deux de maniere equilibree
   - Autre (preciser)
6. **Contraintes** : "Y a-t-il des contraintes particulieres ? (langues, periode temporelle, budget temps, nombre de pistes max, etc.)"

### Phase 2 — Proposition des parametres

A partir des reponses de l'utilisateur, proposer des valeurs adaptees pour chaque section.
Si l'utilisateur n'a pas de preference, utiliser les valeurs par defaut.

Pour chaque section, presenter la valeur proposee et demander :
"Valider ou ajuster ?"

### Phase 3 — Validation section par section

Parcourir chaque section une par une :

#### 3a. Standards de qualite des sources
Proposer un seuil de confiance adapte :
- Recherche rigoureuse (academique) → 70/100
- Recherche equilibree → 60/100 (defaut)
- Veille exploratoire large → 50/100

Presenter les types de sources acceptes et refuses.
Adapter selon le domaine (ex: si industrie, accepter les blogs techniques d'entreprises reconnues).

#### 3b. Grille de scoring multicriteres
Proposer des poids adaptes a l'orientation :
- **Orientation academique** : Serieux /25, Popularite /20, Applicabilite /15, Recence /15, Replicabilite /25
- **Orientation industrielle** : Serieux /15, Popularite /15, Applicabilite /35, Recence /15, Replicabilite /20
- **Equilibree (defaut)** : Serieux /20, Popularite /20, Applicabilite /25, Recence /15, Replicabilite /20

Presenter le bareme detaille de chaque critere.
Les baremes internes (ex: h>=40 → 20/20) sont automatiquement recalcules proportionnellement au poids choisi.

#### 3c. Strategie multi-axes
Adapter selon l'orientation :
- **Academique** : Academique 50-70%, Industriel 15-30%, Public 10-20%
- **Industrielle** : Industriel 50-70%, Academique 20-30%, Public 10-20%
- **Equilibree (defaut)** : Academique 40-60%, Industriel 20-40%, Public 10-30%

Demander si un axe doit etre completement exclu.

#### 3d. Gestion des iterations
Proposer un intervalle de checkpoint adapte :
- Recherche courte (< 30 pistes) → checkpoint toutes les 5 pistes
- Recherche standard → checkpoint toutes les 10 pistes (defaut)
- Recherche longue (> 100 pistes) → checkpoint toutes les 20 pistes

Demander s'il y a un nombre maximum de pistes a explorer.
Demander les conditions d'arret specifiques.

#### 3e. Ethique et biais
Presenter les valeurs par defaut et demander :
- Langues acceptees (defaut: francais, anglais)
- Exigences de diversite specifiques au domaine
- Acteurs a surveiller pour conflits d'interet

#### 3f. Citations et references
Presenter le format BibLaTeX et les champs obligatoires.
Demander si un format de cle de citation different est prefere.

### Phase 4 — Generation

1. Lire le template de ce preset : `.specify/presets/ai-research/templates/constitution-template.md`
   (`specify preset resolve constitution-template` confirme ce chemin). Ne PAS lire
   `.specify/templates/constitution-template.md` : c'est la version core, orientee
   developpement logiciel, que ce preset remplace precisement.
2. Remplir avec toutes les valeurs validees par l'utilisateur
3. Ecrire le resultat dans `.specify/memory/constitution.md`

## Sortie

```
✅ Constitution ecrite dans .specify/memory/constitution.md

Resume :
  Projet : [nom]
  Seuil de confiance : [X]/100
  Scoring : auteurs /[X], popularite /[X], applicabilite /[X], recence /[X], replicabilite /[X]
  Axes : [axe1] [X%] > [axe2] [X%] > [axe3] [X%]
  Checkpoint : toutes les [X] pistes
  Langues : [liste]

→ Prochaine etape : /speckit.specify pour definir votre premier sujet de recherche.
```

## Prochaine etape

Suggerer : `/speckit.specify` pour definir le premier sujet de recherche.
