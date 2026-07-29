# Constitution de recherche — [Nom du projet]

> Principes immuables pour la duree du projet. Etablis via `/speckit.constitution`.

## Standards de qualite des sources

- **Seuil de confiance minimum** : 60/100
- **Sources academiques** : comite de lecture exige (revues, conferences ACM/IEEE/etc.)
- **Sources industrielles** : tracabilite exigee (auteur identifie, entreprise, date)
- **Sources publiques** : organisme officiel ou institution reconnue
- **Sources refusees** : blogs anonymes, forums non moderes, sources sans date

## Grille de scoring multicriteres

| Critere | Poids | Source de verification | Bareme |
|---------|-------|-----------------------|--------|
| Serieux des auteurs | /20 | OpenAlex (h-index) | h>=40: 20, h>=25: 16, h>=15: 12, h>=8: 8, h>=3: 4, h<3: 2 |
| Popularite de la source | /20 | OpenAlex (citations) | >=500: 20, >=200: 16, >=50: 12, >=10: 8, >=1: 4, 0: 2 |
| Applicabilite | /25 | Analyse du contenu | Produit: 25, Demo: 20, Proto: 15, Methodo: 10, Theorie: 5 |
| Recence | /15 | Date de publication | 0-1an: 15, 2-3ans: 12, 4-5ans: 8, 6-8ans: 4, >8ans: 2 |
| Replicabilite | /20 | Disponibilite ressources | Code+data+methodo: 20, Code+methodo: 16, Methodo: 12, Partiel: 8, Rien: 4 |

**Total : /100**

## Strategie multi-axes

| Axe | Cible | Seuil d'alerte |
|-----|-------|----------------|
| Academique | 40-60% | < 20% |
| Industriel | 20-40% | < 10% |
| Public/Institutionnel | 10-30% | < 5% |

Priorite par defaut : academique > industriel > public (ajustable par sujet).

## Gestion des iterations

- **Intervalle entre checkpoints** : 10 pistes explorees
- **Conditions d'arret** :
  - Toutes les questions P1 couvertes avec score >= seuil
  - Saturation (5 pistes consecutives sans nouvelle idee)
  - Decision utilisateur
- **Re-evaluation** : autorisee a tout moment via `/speckit.research.score`

## Ethique et biais

- **Diversite des sources** : minimum 2 axes representes par question de recherche
- **Biais de confirmation** : explorer activement les sources qui contredisent les premiers resultats
- **Langues acceptees** : francais, anglais (extensible)
- **Conflits d'interet** : signaler si une source est produite par un acteur interesse

## Citations et references

- **Format** : BibLaTeX
- **Champs obligatoires** : author, title, year, url, urldate, file, keywords, note
- **Politique de telechargement** : copie locale dans `sources/[axe]/` quand possible
- **Cle de citation** : `auteur-annee-motcle` (ex: `smith-2024-llm-agents`)
