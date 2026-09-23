# Prévision du rendement des agrumes — Compétition Kaggle

Ce dépôt contient le travail réalisé pour une compétition Kaggle de **prévision du tonnage
mensuel récolté par parcelle** dans des vergers d'agrumes (clémentiniers, orangers, mandariniers...)
répartis sur plusieurs régions du Maroc (Gharb, Haouz, Souss, Tadla, Oriental).

L'objectif : à partir de l'historique de production, des données d'irrigation, de fertilisation
(NPK) et de climat sur les campagnes 2020/21 à 2023/24, prédire le tonnage récolté mois par mois
et parcelle par parcelle pour la campagne 2024/25 (totalement absente de l'entraînement).

> Les données brutes et les documents de cadrage ne sont **pas inclus** dans ce dépôt (voir
> [Données](#données) ci-dessous) : ce README ne reproduit aucune information nominative ou
> propriétaire (noms de domaines réels, chiffres internes de production, etc.).

## Structure du projet

```
.
├── Data/
│   ├── train.csv                # Historique de tonnage (2020/21 → 2023/24) — fourni par la compétition
│   ├── test.csv                 # Lignes à prédire (campagne 2024/25) — fourni par la compétition
│   └── ...                      # Tables auxiliaires (irrigation, NPK, parcellaire, climat) — non versionnées
├── notebooks/
│   ├── 01_data_preparation.ipynb        # Chargement, nettoyage, jointures, feature engineering temporel
│   ├── 02_exploratory_analysis.ipynb    # Analyse exploratoire sur les données nettoyées
│   ├── 03_feature_engineering.ipynb     # Encodage, transformations, construction des matrices modèle
│   └── 04_modeling_and_submission.ipynb # Validation, entraînement, génération de la soumission
└── submission.csv               # Fichier de soumission généré par le notebook 04
```

Chaque notebook a un rôle unique et se relit indépendamment : le 01 produit une table maître
propre, le 02 l'explore, le 03 en dérive les features finales, le 04 entraîne et évalue le modèle
puis écrit `submission.csv`. Ils s'exécutent dans l'ordre (chacun consomme les fichiers produits
par le précédent, sauvegardés dans `Data/processed/`).

## Démarche

1. **Préparation des données** — construction d'une table au grain *parcelle × mois de récolte*,
   avec une clé de jointure composite robuste (une clé technique fournie dans les données brutes
   s'est révélée non unique et ne doit jamais servir à agréger), correction des valeurs de saisie
   aberrantes, et calcul de l'âge du verger par rapport à la date de récolte réelle de chaque ligne.
2. **Alignement temporel des intrants** — l'irrigation et les apports d'engrais sont résumés en
   cumuls glissants sur les mois *précédant* la récolte (et non le mois de la récolte lui-même),
   pour refléter le décalage biologique entre l'apport et son effet sur le fruit.
3. **Climat** — les données climatiques régionales sont rattachées au niveau de chaque site de
   production via une table de correspondance, avec repli sur la moyenne régionale quand un site
   ne dispose pas de sa propre station météo.
4. **Feature engineering** — transformations non linéaires de l'âge, ratios agronomiques (azote/
   potassium, apports par surface), encodage des variables catégorielles par moyenne de cible
   calculée hors échantillon (sans fuite de données), variable d'historique de production
   (rendement de la campagne précédente).
5. **Modélisation et validation** — un modèle de gradient boosting (LightGBM) est comparé à une
   baseline simple, et évalué à la fois par **validation temporelle** (entraînement sur les
   campagnes passées, test sur la dernière campagne connue) et par **validation par groupe**
   (parcelles jamais vues), pour mesurer séparément la capacité d'extrapolation dans le temps et
   la généralisation à de nouvelles parcelles.

## Données

Le dossier `Data/` n'est versionné que pour `train.csv` et `test.csv` (fichiers fournis par la
compétition Kaggle). Les tables auxiliaires (irrigation, fertilisation, parcellaire, climat, ainsi
que les fichiers intermédiaires générés dans `Data/processed/`) sont ignorées par Git — voir
`.gitignore` — car elles contiennent des informations propres à l'exploitation agricole
d'origine. 
