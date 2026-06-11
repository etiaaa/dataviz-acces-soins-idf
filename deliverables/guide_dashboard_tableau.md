# Guide de construction du dashboard Tableau Public

Objectif : 2 niveaux de zoom — **vue EPCI** (président d'intercommunalité) et **vue commune** (maire).
Source principale : `data/processed/communes_idf_score.geojson` (1 ligne = 1 commune, score + KPIs inclus).

---

## 0. Connexion des données

1. Tableau Public → **Connexion → Fichier spatial** → choisir `communes_idf_score.geojson`.
2. Tableau crée automatiquement un champ **Geometry** + un champ par propriété :
   `code_insee, nom_commune, dep, epci_nom, epci_nature, pop_commune, apl_generaliste,
   densite_cardio_100k, densite_dermato_100k, densite_ophtalmo_100k, densite_gyneco_100k,
   tension_specialistes, score_tension, rang_epci_pct`.
3. Vérifier les types : `score_tension`, `apl_generaliste`, les `densite_*` = **mesures (nombre décimal)** ;
   `code_insee, nom_commune, dep, epci_nom` = **dimensions**.

> Si la carte ne se trace pas : clic droit sur **Geometry** → *Rôle géographique → Géométrie*.

---

## 1. Feuille « Carte EPCI » (vue président)

But : voir tout le territoire colorié par tension, repérer les communes prioritaires.

- **Geometry** → dans la vue (la carte apparaît).
- **Couleur** : `score_tension` (agrégation **AVG**, mais 1 ligne/commune donc = la valeur).
  Palette divergente inversée : vert (bas) → rouge (haut). Plage fixe **0–100**.
- **Info-bulle** : `nom_commune`, `score_tension`, `apl_generaliste`, `tension_specialistes`, `rang_epci_pct`, `pop_commune`.
- **Filtre** : `epci_nom` (afficher le filtre → le président choisit son intercommunalité).
- Option : **Détail** = `code_insee` pour garder une commune par polygone.

Lecture : une fois filtré sur un EPCI, les communes les plus rouges = là où mutualiser (MSP, télémédecine…).

---

## 2. Feuille « Profil commune » (vue maire)

But : une commune sélectionnée, ses KPIs comparés à la moyenne EPCI et régionale.

### a) Indicateurs de la commune (cartes de score / BANs)
- Texte simple : `score_tension`, `apl_generaliste`, et les 4 `densite_*`.
- Filtre `nom_commune` (ou action depuis la carte, voir §4).

### b) Comparaison à la moyenne EPCI et régionale (champs calculés)
Créer ces **champs calculés** (LOD) :

```
APL moyen EPCI      = { FIXED [Epci Nom] : AVG([Apl Generaliste]) }
APL moyen région    = { FIXED : AVG([Apl Generaliste]) }
Score moyen EPCI    = { FIXED [Epci Nom] : AVG([Score Tension]) }
Écart APL vs EPCI   = AVG([Apl Generaliste]) - [APL moyen EPCI]
```

- Barres comparatives : commune vs `APL moyen EPCI` vs `APL moyen région`.
- `Écart APL vs EPCI` en couleur (rouge si négatif = en dessous de ses voisines).

### c) Position relative dans l'EPCI
- `rang_epci_pct` affiché en jauge (0 = la mieux dotée de l'EPCI, 1 = la plus tendue).
  Mise en forme en %.

---

## 3. Feuille « Méthodo » (onglet explicatif)
Texte : sources (DREES, RPPS, INSEE, BANATIC), définition de l'APL, choix des 4 spécialités
(les plus tendues — Doctolib/Jean-Jaurès 2026), construction du score (½ généralistes + ½ spécialistes,
normalisation min-max winsorisée, inversion = tension). Mentionner la limite : densité spécialistes
au département, KPI âge à venir.

---

## 4. Assemblage du dashboard
- Nouveau **Dashboard**, format paysage (ex. 1366×768).
- Disposer : titre + « Carte EPCI » à gauche, « Profil commune » à droite, filtre `epci_nom` en haut.
- **Action de filtre** : *Tableau de bord → Actions → Ajouter → Filtre* ; source = Carte EPCI,
  cible = Profil commune, exécuter **au clic** sur `code_insee`.
  → cliquer une commune sur la carte met à jour son profil. C'est le lien entre les 2 niveaux de zoom.
- **Action de surbrillance** sur `epci_nom` pour faire ressortir l'intercommunalité.

---

## 5. Publication
- *Serveur → Tableau Public → Enregistrer sur le Web* (compte Tableau Public gratuit requis).
- Le `.geojson` est embarqué (extrait) → le dashboard reste autonome.
- Récupérer le lien public pour la note méthodo et l'oral.

---

## Bonus — coloration « maline » des barres (au-dessus / en-dessous de la moyenne IDF)

Pour colorer chaque barre en **vert si la commune est au-dessus de la moyenne régionale**, **rouge si en-dessous** (le vrai bon/mauvais par indicateur) :

1. Créer 5 champs calculés (menu **Analyse → Créer un champ calculé**). Astuce : taper les noms de champs via l'autocomplétion, et le `-` avec la touche clavier (pas de copier-coller, qui insère parfois un faux tiret) :

```
Écart Cardio    = AVG([Densite Cardio 100K])   - {FIXED : AVG([Densite Cardio 100K])}
Écart Dermato   = AVG([Densite Dermato 100K])  - {FIXED : AVG([Densite Dermato 100K])}
Écart Ophtalmo  = AVG([Densite Ophtalmo 100K]) - {FIXED : AVG([Densite Ophtalmo 100K])}
Écart Gyneco    = AVG([Densite Gyneco 100K])   - {FIXED : AVG([Densite Gyneco 100K])}
Écart APL       = AVG([Apl Generaliste])       - {FIXED : AVG([Apl Generaliste])}
```

2. Sur la feuille `Profil commune`, remplacer les 5 mesures de l'encadré **Valeurs de mesures** par ces 5 champs `Écart …`.
3. Glisser **Valeurs de mesures** sur **Couleur** → *Modifier les couleurs* → palette **Rouge-Vert divergent**, **Centre = 0** (vert = au-dessus de la moyenne = bon, rouge = en-dessous = mauvais).

→ Les barres partent de 0 : vers la droite (vert) = mieux que la moyenne IDF ; vers la gauche (rouge) = moins bien. Lecture immédiate du bon/mauvais par spécialité.

### Rappels storytelling (pour l'oral)
- Accroche : « 62 % des communes franciliennes sous le seuil de sous-dotation en généralistes. »
- Contraste fort : dermatologues **×13** entre Paris et la Seine-Saint-Denis.
- Message aux élus : prioriser **localement** (rang intra-EPCI), pas un palmarès national.
