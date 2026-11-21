# TP4 – Analyse des données et architecture Azure

## Présentation

Dans ce TP4, j'ai travaillé sur le projet Retail Intelligence Platform avec les données TPCx-BB fournies dans le cours.
## Outils utilisés
- Python et PySpark pour les traitements
- JupyterLab pour exécuter le code
- Pandas et Matplotlib pour les graphiques
- Docker pour l'environnement de travail
- Microsoft Azure avec ADLS Gen2 pour le stockage
- Git et GitHub pour enregistrer le projet

## Travail réalisé

### UC-08 – Analyse des données

J'ai commencé par charger les données TPCx-BB dans JupyterLab.

J'ai ensuite préparé les données en utilisant l'organisation Bronze, Silver et Gold :

- **Bronze** : données brutes
- **Silver** : données préparées et vérifiées
- **Gold** : résultats regroupés pour les analyses

J'ai travaillé principalement sur les ventes en magasin avec 667 579 lignes de données.

J'ai calculé trois indicateurs : le montant total des ventes, le montant moyen par ligne et le nombre de produits différents.

J'ai également réalisé des graphiques pour comparer les ventes par magasin, voir leur évolution dans le temps et identifier les produits qui représentent les montants de ventes les plus élevés.

### UC-07 – Architecture Azure

Pour cette partie, j'ai utilisé Azure pour créer un espace de stockage ADLS Gen2.

J'ai créé un conteneur `retail-intelligence` avec des dossiers Bronze, Silver et Gold.

J'ai déposé un petit échantillon du fichier `store_sales.dat` dans Bronze et vérifié que je pouvais consulter son contenu.

J'ai aussi découvert les permissions RBAC et ACL, ainsi que le rôle des services Azure Data Factory et Synapse Analytics.

Les traitements Silver et Gold ont été réalisés avec PySpark en local. Sur Azure, j'ai créé les dossiers correspondants, mais je n'ai pas envoyé les fichiers Parquet.

## Fichiers du projet

- **`tp4.ipynb`** : notebook avec le code Python, les calculs et les graphiques.
- **`TP4_rapport.md`** : compte rendu du TP avec les résultats et les explications.
- **`README.md`** : présentation du projet.

## Conclusion

Ce TP m'a permis de mieux comprendre comment préparer des données avec PySpark, calculer des indicateurs et réaliser des graphiques.

J'ai aussi découvert le fonctionnement d'un Data Lake sur Azure et la différence entre les zones Bronze, Silver et Gold.

