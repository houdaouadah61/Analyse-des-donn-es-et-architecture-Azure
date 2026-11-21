# TP4 – Analyse des données et architecture Azure

## Introduction

Dans ce TP, j'ai travaillé sur le projet *Retail Intelligence Platform*. Le but était d'analyser les données de ventes TPCx-BB et de découvrir comment organiser ces données dans un environnement Azure.

Le travail comporte deux parties. La première concerne l'analyse des données avec Python et PySpark (UC-08). La deuxième concerne la mise en place d'une architecture de stockage sur Azure (UC-07).

J'ai utilisé JupyterLab dans un environnement Docker pour les traitements, puis Microsoft Azure pour la partie Data Lake.

## 1. UC-08 – Analyse des données et visualisation

### 1.1. Données utilisées

Pour commencer, j'ai utilisé les fichiers TPCx-BB fournis avec le projet. Ce sont des données synthétiques qui permettent de simuler l'activité commerciale d'une entreprise.

J'ai principalement travaillé avec trois tables :

- `store_sales` : les ventes réalisées en magasin ;
- `item` : les informations sur les produits ;
- `date_dim` : les informations sur les dates.

Les données de départ sont au format `.dat`, avec des valeurs séparées par le caractère `|`.

La table `store_sales` contient **667 579 lignes**.

### 1.2. Préparation des données Silver et Gold

Dans une architecture Data Lake, les données sont organisées en trois zones :

- **Bronze** : les données brutes, telles qu'elles arrivent dans le système.
- **Silver** : les données préparées et contrôlées pour pouvoir les utiliser.
- **Gold** : les données finales, souvent regroupées sous forme d'indicateurs pour l'analyse.

Pour préparer les données Silver, j'ai utilisé PySpark et sélectionné les colonnes utiles : `date_id`, `produit_id`, `magasin_id` et `montant_vente`.

J'ai ensuite vérifié les valeurs manquantes et les montants négatifs. Les résultats étaient de **0 dans les deux cas**. Les 667 579 lignes ont donc été conservées.

J'ai enregistré les données Silver au format Parquet.

Pour Gold, j'ai regroupé les ventes par magasin afin d'obtenir le montant total des ventes de chaque magasin. Le résultat contient 12 magasins et a également été enregistré en Parquet.

Les fichiers Parquet Silver et Gold ont été produits dans mon environnement local avec PySpark. Sur Azure, j'ai créé les dossiers correspondants, mais je n'y ai pas transféré ces fichiers.

### 1.3. Calcul des KPI

J'ai calculé trois indicateurs à partir des données Silver.

| Indicateur | Calcul | Résultat |
|---|---|---|
| Montant total des ventes | Somme de `montant_vente` | 1 150 529 586,37 |
| Montant moyen par ligne | Moyenne de `montant_vente` | 1 723,44 |
| Nombre de produits différents | Nombre de `produit_id` distincts | 17 820 |

Ces calculs portent sur l'ensemble des lignes sélectionnées, sans filtre supplémentaire.

Les montants sont exprimés dans l'unité monétaire du jeu de données, qui n'a pas été précisée. Le nombre de produits est un nombre d'identifiants distincts.

La moyenne par ligne ne correspond pas forcément au panier moyen d'un client, car une ligne de vente n'est pas nécessairement une transaction complète.

### 1.4. Statistiques descriptives

J'ai aussi calculé quelques statistiques pour mieux comprendre la répartition des montants.

| Statistique | Résultat |
|---|---|
| Nombre de lignes | 667 579 |
| Montant minimum | 0 |
| Montant maximum | 18 745 |
| Moyenne | 1 723,44 |
| Médiane estimée | 863,30 |

On remarque que la moyenne est plus élevée que la médiane. Cela peut s'expliquer par la présence de ventes avec des montants importants qui augmentent la moyenne.

Il y a aussi des montants égaux à zéro. Ce n'est pas forcément une erreur, mais il faudrait connaître leur signification pour aller plus loin.

### 1.5. Analyses et graphiques

J'ai réalisé trois graphiques avec Matplotlib dans mon notebook.

**Comparaison des ventes par magasin**

Pour cette analyse, j'ai regroupé les données `store_sales` par identifiant de magasin, puis calculé la somme des montants de vente.

Le magasin 7 arrive en première position avec **98 950 137,76**. Le magasin 2 a un montant de **94 040 402,78**.

Le graphique en barres permet de comparer facilement les 12 magasins. Les écarts existent, même s'ils restent relativement proches.

On ne peut cependant pas expliquer ces différences sans informations supplémentaires sur les magasins, comme leur taille ou leur fréquentation.

**Évolution mensuelle des ventes**

J'ai utilisé `store_sales` et `date_dim` pour regrouper les ventes par mois. J'ai ensuite représenté les résultats avec une courbe.

Le graphique montre l'évolution des montants de ventes entre 2001 et 2005. Les montants varient selon les mois.

Cette représentation permet de repérer les périodes où les ventes sont plus élevées ou plus faibles. En revanche, une variation ne suffit pas à expliquer sa cause, et il faudrait vérifier que toutes les périodes contiennent des données complètes.

**Top 10 des produits**

J'ai regroupé les ventes par identifiant de produit et calculé le montant total de chacun.

Le produit **10000** arrive en première position avec **15 408 393,55**, suivi du produit **10003** avec **14 933 870,10**.

J'ai représenté les dix premiers produits avec un diagramme en barres. J'ai utilisé les identifiants des produits, car les noms présents dans le fichier n'étaient pas facilement lisibles.

Cette analyse montre les produits qui représentent les montants de ventes les plus importants, mais pas forcément ceux qui sont vendus en plus grande quantité.

Les trois graphiques et leurs résultats sont visibles dans le notebook `tp4.ipynb`.

### 1.6. Constats

Le premier constat concerne les magasins. Le magasin 7 réalise le montant de ventes le plus important. Cela peut montrer une différence d'activité entre les magasins, mais les données ne permettent pas de connaître la raison exacte. Pour approfondir, il serait intéressant de comparer aussi les quantités vendues et les caractéristiques de chaque magasin.

Le deuxième constat concerne les produits. Le produit 10000 contribue fortement au montant total des ventes. Il pourrait donc être intéressant d'étudier ses quantités vendues et sa répartition entre les magasins. Il faut cependant éviter de conclure qu'il s'agit du produit le plus populaire uniquement à partir du montant des ventes.

Ces constats restent liés à des données synthétiques et ne représentent pas la situation réelle d'une entreprise.

## 2. UC-07 – Architecture Azure

### 2.1. Mise en place du stockage

Pour cette partie, j'ai choisi le parcours Azure proposé dans le TP.

J'ai utilisé mon compte étudiant pour créer un groupe de ressources et un compte de stockage Azure.

Dans ce compte, j'ai créé un conteneur nommé `retail-intelligence`, avec une organisation Bronze, Silver et Gold.

La structure utilisée est la suivante :

```text
retail-intelligence/
    bronze/
        tpcxbb/
    silver/
        ventes/
        produits/
    gold/
        indicateurs/
        analyses/
```

Cette organisation permet de séparer les données selon leur niveau de préparation.

Dans Bronze, on conserve les fichiers sources. Silver sert aux données nettoyées ou préparées. Gold est destiné aux résultats et aux indicateurs utilisés pour les analyses.

Pour les fichiers, j'ai conservé des noms simples qui indiquent leur contenu. Dans une utilisation plus complète, il faudrait aussi prévoir le suivi des versions, des dates de traitement et la conservation des fichiers sources.

### 2.2. Azure Data Lake Storage Gen2

J'ai utilisé Azure Data Lake Storage Gen2 (ADLS Gen2) pour organiser le stockage.

J'ai également vérifié le **Hierarchical Namespace (HNS)**. Cette fonctionnalité permet de travailler avec de vrais répertoires et sous-répertoires, et de gérer les permissions sur les dossiers et les fichiers.

C'est notamment utile pour organiser un projet en zones Bronze, Silver et Gold. Par rapport à un stockage objet classique sans HNS, les opérations sur les chemins et les autorisations des répertoires sont mieux adaptées à ce type d'organisation.

### 2.3. Test du stockage Azure

Pour vérifier que le stockage fonctionnait, j'ai préparé un échantillon de 100 lignes à partir de `store_sales.dat`.

Le fichier créé s'appelle `echantillon_store_sales.dat` et fait 13 394 octets.

Je l'ai envoyé dans le chemin :

`retail-intelligence/bronze/tpcxbb/echantillon_store_sales.dat`

J'ai ensuite ouvert le fichier dans Azure pour vérifier que les données étaient bien présentes.

Azure affichait un avertissement concernant l'extension `.dat`, mais le contenu du fichier restait visible.

J'ai gardé des captures d'écran du fichier et de ses informations pour montrer la manipulation réalisée.

### 2.4. Sécurité et droits d'accès

J'ai aussi consulté les paramètres de sécurité du compte de stockage.

Dans Azure, j'ai trouvé le rôle **Lecteur** associé à mon accès. Ce rôle permet de consulter les ressources Azure, mais ne donne pas automatiquement tous les droits sur les données.

J'ai ensuite regardé les permissions ACL du dossier `bronze`.

| Identité | Permissions |
|---|---|
| Propriétaire | Lecture, écriture et exécution |
| Groupe propriétaire | Lecture et exécution |
| Autres | Aucune |

Les permissions observées correspondent à `rwxr-x---`.

J'ai retenu le principe du moindre privilège : chaque utilisateur doit avoir uniquement les droits nécessaires.

Pour notre projet, la répartition proposée est la suivante :

| Profil | Bronze | Silver | Gold |
|---|---|---|---|
| Ingestion | Écriture contrôlée | Lecture limitée | Aucun accès |
| Data Engineer | Lecture et écriture | Lecture et écriture | Écriture contrôlée |
| Analyste | Aucun accès | Lecture si nécessaire | Lecture |
| Administrateur | Selon ses habilitations | Selon ses habilitations | Selon ses habilitations |

Il s'agit d'une proposition de droits. Je n'ai pas créé de nouveaux utilisateurs ni appliqué cette matrice sur Azure.

Microsoft Entra ID sert à gérer les identités et l'authentification. Azure RBAC gère les rôles sur les ressources et les ACL permettent de définir des droits plus précis sur les fichiers et les dossiers.

Les mots de passe, les clés et les tokens d'accès doivent rester privés et ne pas être ajoutés au dépôt GitHub.

### 2.5. Data Factory, Synapse et alternatives

J'ai étudié les services Azure qui pourraient être utilisés pour compléter cette architecture.

| Service Azure | Utilité | Alternative locale |
|---|---|---|
| ADLS Gen2 | Stockage des données | MinIO ou dossiers locaux |
| Data Factory | Ingestion et automatisation | Airflow ou scripts |
| Synapse Spark | Traitement des données | PySpark |
| Synapse SQL | Requêtes SQL | Trino ou Spark SQL |
| Tables analytiques | Organisation des tables | Iceberg et Parquet |
| Services de métadonnées | Gestion des métadonnées | Nessie ou Hive Metastore |
| Power BI | Visualisation | Matplotlib ou Superset |

Dans mon TP, les traitements ont été réalisés avec PySpark en local.

Azure Data Factory et Synapse ont seulement été étudiés. Je n'ai pas créé de pipeline Data Factory ni de workspace Synapse.

La différence est qu'Azure propose plusieurs services gérés dans le cloud, alors qu'avec des solutions open source, il faut souvent installer et relier plusieurs outils.

Par exemple, Iceberg sert à gérer des tables et leurs métadonnées, mais ne remplace pas à lui seul Synapse. Il faut également un moteur comme Spark pour les traitements et un outil SQL pour les requêtes.

### 2.6. Architecture retenue

Le fonctionnement général du projet peut être représenté ainsi :

```text
           Données TPCx-BB
                  |
          JupyterLab / PySpark
                  |
         Préparation des données
                  |
          Silver (Parquet local)
                  |
          Gold (Parquet local)
                  |
       KPI et graphiques Matplotlib


           Données TPCx-BB
                  |
       Dépôt manuel d'un échantillon
                  |
           Azure ADLS Gen2
                  |
               Bronze
                  |
        Silver et Gold prévus
```

Les traitements et les graphiques ont réellement été exécutés dans JupyterLab. Sur Azure, j'ai réellement créé le stockage et les répertoires, puis testé le dépôt dans Bronze.

La circulation automatique des données avec Data Factory et les traitements avec Synapse restent des possibilités étudiées, pas des fonctionnalités déployées.

