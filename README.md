# Accès aux soins en Île-de-France — où agir en priorité ?

Tableau de bord d'aide à la décision pour les élus locaux (maires et présidents d'EPCI),
pour identifier **où agir** (carte par EPCI) et **sur quoi agir** (profil de chaque commune)
face à la tension d'accès aux soins.

Projet de validation — Data Visualisation, Ynov — SAKOA Etia-Anaëlle.

## Dashboard en ligne

➡️ **[Voir le tableau de bord sur Tableau Public](https://public.tableau.com/app/profile/etia.ana.lle.sakoa/viz/Accsauxsoinsenle-de-Franceoagirenpriorit/Tableaudebord)**

Deux vues complémentaires :
- **Carte EPCI** — choroplèthe des 1 266 communes franciliennes colorées par score de tension.
- **Ma commune vs moyenne IDF** — comparaison des indicateurs d'une commune à la moyenne régionale.

## Le score de tension

Un indice unique de 0 à 100 par commune (plus il est haut, plus l'accès est tendu),
construit sur **3 dimensions à parts égales** :

| Dimension | Indicateur | Unité |
|---|---|---|
| Généralistes | APL (accessibilité potentielle localisée) | consultations / hab / an |
| Spécialistes | densité de 4 spécialités (cardio, dermato, ophtalmo, gynéco) | médecins / 100 000 hab |
| Démographie | âge moyen des médecins | années |

Chaque indicateur est normalisé sur 0–100 (min-max winsorisé aux centiles 2/98).
Les indicateurs d'offre sont inversés (offre faible = tension forte) ; l'âge est en direct
(médecins âgés = renouvellement à risque). Score observé : de 17 (mieux dotée) à 93 (la plus tendue).

## Sources de données (open data, 2023–2025)

- **DREES** — APL généralistes 2023 ; densité des spécialistes (RPPS) 2024 ; âge moyen des médecins 2025.
- **INSEE** — périmètre des communes (COG).
- **BANATIC** — rattachement commune → EPCI et population municipale.

> Densité des spécialistes et âge des médecins disponibles au niveau **département**.

## Structure du dépôt

```
project_dashboard_health_access/
├── subject/                     # sujet du projet
├── deliverables/
│   ├── 02_nettoyage_jointures.ipynb   # consolidation + jointures par code INSEE
│   ├── 03_eda.ipynb                   # analyse exploratoire
│   ├── 04_score_tension.ipynb         # construction du score de tension
│   ├── note_methodologique.ipynb      # note méthodologique
│   ├── guide_dashboard_tableau.md     # guide de construction du dashboard
│   └── data/
│       ├── raw/                       # données sources (DREES, INSEE, BANATIC, GeoJSON)
│       └── processed/                 # données consolidées (score, geojson, comparaison)
└── Acces-aux-soins-en-Ile-de-France-ou-agir-en-priorite.pdf   # support de présentation
```

## Reproduire

Exécuter les notebooks dans l'ordre `02` → `03` → `04`.
Le `04` régénère les fichiers de `data/processed/` qui alimentent le dashboard Tableau.

Librairies : `pandas`, `openpyxl`, `pyarrow`.
