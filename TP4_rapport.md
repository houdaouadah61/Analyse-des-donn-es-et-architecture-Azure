
# TP4 – Analyse des données et architecture Azure

**Projet : Retail Intelligence Platform**  
**UC-08 : Analyse des données et visualisation**  
**UC-07 : Architecture Azure**

## Introduction

Dans ce quatrième TP, j'ai travaillé sur les données TPCx-BB fournies avec le projet Retail Intelligence Platform.

Le but était de réutiliser les données pour calculer des indicateurs, faire des graphiques et comprendre comment on peut organiser les données dans un Data Lake sur Azure.

J'ai commencé par travailler sur les fichiers avec PySpark dans JupyterLab, puis j'ai réalisé les analyses et les visualisations. Ensuite, j'ai utilisé Azure pour créer les zones Bronze, Silver et Gold et tester le stockage d'un fichier.

Comme c'était ma première fois avec plusieurs de ces outils, j'ai avancé étape par étape pour comprendre leur fonctionnement.

## 1. Environnement de travail

### 1.1 Docker et JupyterLab

Pour réaliser ce TP, j'ai utilisé Docker avec un environnement Linux contenant les outils Python nécessaires.

Mon conteneur de travail s'appelait `tp4-final`.

J'accédais à JupyterLab depuis le navigateur avec l'adresse :

`http://localhost:8890/lab`

Le port **8890** permettait donc d'accéder à JupyterLab depuis mon ordinateur.

Dans le notebook, j'ai vérifié que le dossier de travail était `/workspace` et que Python était exécuté depuis `/usr/local/bin/python3.13`.

Les outils utilisés étaient :

- **PySpark** pour lire et traiter les données.
- **Pandas** pour récupérer les résultats des calculs dans des petits tableaux.
- **Matplotlib** pour créer les graphiques.
- **JupyterLab** pour écrire et exécuter le code.
- **Docker** pour faire fonctionner l'environnement.

J'ai créé une session Spark en mode local avec `master("local[*]")`.

Les traitements ont donc été exécutés dans Docker sur mon ordinateur, et pas directement sur Azure.

### 1.2 Vérification des fichiers

J'ai commencé par regarder les fichiers du projet.

L'archive utilisée était `retail_project (1).zip`, avec les données extraites dans :

`/workspace/Projet 4/retail_project`

Dans les dossiers du projet, j'ai trouvé 23 tables dans `data` et 11 tables dans `data_refresh`.

J'ai principalement utilisé les tables suivantes :

| Table | Nombre de colonnes | Utilisation |
|---|---:|---|
| `store_sales` | 23 | Analyse des ventes en magasin |
| `web_sales` | 34 | Vérification des données disponibles |
| `item` | 22 | Informations sur les produits |
| `date_dim` | 28 | Analyse des ventes par date |

Les fichiers étaient au format `.dat`, avec le séparateur `|`.

Comme ils n'avaient pas d'en-têtes, Spark affichait les colonnes avec des noms comme `_c0`, `_c2` ou `_c20`.

J'ai donc consulté les premières lignes et les schémas avant de choisir les colonnes nécessaires pour les analyses.

## 2. UC-08 – Analyse des données et visualisation

### 2.1 Préparation des données Silver et Gold

Pour cette partie, j'ai repris l'organisation des données étudiée en cours :

- **Bronze :** les données brutes, sans transformation.
- **Silver :** les données préparées et vérifiées.
- **Gold :** les données finales utilisées pour les indicateurs et les analyses.

Les fichiers `.dat` fournis correspondent aux données brutes.

Pour préparer les données Silver, j'ai utilisé la table `store_sales` et sélectionné les colonnes utiles.

| Colonne Silver | Colonne d'origine | Utilisation |
|---|---|---|
| `date_id` | `_c0` | Identifiant de la date |
| `produit_id` | `_c2` | Identifiant du produit |
| `magasin_id` | `_c7` | Identifiant du magasin |
| `montant_vente` | `_c20` | Montant des ventes |

J'ai obtenu **667 579 lignes**.

J'ai ensuite vérifié les valeurs manquantes dans les quatre colonnes et les montants négatifs.

Les résultats étaient :

- Valeurs manquantes : **0**
- Montants négatifs : **0**

J'ai donc conservé les lignes pour les analyses. Cela ne veut pas dire que les données sont parfaites, mais les contrôles réalisés n'ont pas montré de problème sur ces points.

Les données Silver ont été enregistrées en Parquet dans :

`/workspace/Projet 4/retail_project/silver/ventes`

Pour Gold, j'ai regroupé les ventes par magasin et calculé la somme des montants.

J'ai obtenu **12 lignes**, correspondant aux 12 identifiants de magasins.

Le résultat Gold a été enregistré dans :

`/workspace/Projet 4/retail_project/gold/indicateurs`

Ces deux fichiers Parquet ont été créés localement avec PySpark. Sur Azure, j'ai aussi créé les dossiers Silver et Gold, mais je n'y ai pas transféré les fichiers Parquet.

### 2.2 Calcul des KPI

J'ai calculé trois indicateurs à partir des données Silver.

| KPI | Calcul | Résultat |
|---|---|---:|
| Montant total des ventes | Somme de `montant_vente` | 1 150 529 586,37 |
| Montant moyen par ligne | Moyenne de `montant_vente` | 1 723,44 |
| Nombre de produits différents | Nombre de `produit_id` distincts | 17 820 |

Ces indicateurs ont été calculés sur toutes les lignes de `silver_ventes`, sans appliquer de filtre particulier.

Le montant moyen correspond à la moyenne des lignes de vente. Ce n'est pas un panier moyen, car une ligne ne représente pas forcément une transaction complète.

De même, les 17 820 produits représentent des identifiants distincts et non une quantité totale vendue.

L'unité monétaire n'étant pas précisée dans les résultats du jeu de données, je n'ai pas considéré ces montants comme des euros.

### 2.3 Statistiques descriptives

J'ai ensuite calculé quelques statistiques sur les montants des ventes.

| Statistique | Résultat |
|---|---:|
| Nombre de lignes | 667 579 |
| Montant minimum | 0 |
| Montant maximum | 18 745 |
| Moyenne | 1 723,44 |
| Médiane approximative | 863,30 |

La médiane a été calculée avec `approxQuantile`, donc le résultat est approximatif.

J'ai remarqué que la moyenne est presque deux fois plus élevée que la médiane.

Cela peut s'expliquer par la présence de ventes avec des montants importants qui augmentent la moyenne.

J'ai aussi remarqué que le montant minimum était égal à zéro. Il faudrait vérifier à quoi correspondent ces lignes avant de décider si elles doivent être conservées ou non.

### 2.4 Analyse des graphiques

Pour la visualisation, j'ai réalisé trois graphiques avec Matplotlib.

J'ai d'abord utilisé PySpark pour faire les regroupements, puis Pandas pour récupérer les résultats et les afficher.

Les graphiques sont disponibles dans le notebook `tp4.ipynb`.

#### 2.4.1 Graphique 1 – Comparaison du montant total des ventes par magasin

**Objectif :** comparer les montants des ventes entre les différents magasins.

Pour réaliser ce graphique, j'ai utilisé `store_sales`.

J'ai regroupé les lignes par identifiant de magasin, puis calculé la somme des montants de vente.

J'ai obtenu un graphique en barres représentant les 12 magasins.

**Résultats observés**

Le magasin **7** arrive en première position avec un montant de **98 950 137,76**.

Il est suivi notamment des magasins 4 et 9, dont les montants sont proches de 96,7 millions.

Le magasin **2** présente le montant le plus faible avec **94 040 402,78**.

L'écart entre le magasin 7 et le magasin 2 est d'environ **4,91 millions**, soit environ **5,2 %** du montant du magasin 2.

**Analyse**

En regardant le graphique, je remarque que les montants des ventes sont assez proches entre les magasins.

Le magasin 7 est celui qui réalise le montant le plus élevé, mais il n'y a pas une très grande différence avec les autres.

Je peux donc dire que les ventes sont relativement réparties entre les magasins dans ce jeu de données.

Par contre, je ne peux pas affirmer que le magasin 7 est réellement plus performant. Il faudrait connaître d'autres informations, comme la taille du magasin, le nombre de clients ou les quantités vendues.

J'ai aussi remarqué la présence d'un magasin avec l'identifiant `0`. Je n'ai pas vérifié sa signification dans cette analyse.

**Source :** `store_sales`, somme des montants `_c20` par magasin `_c7`.

#### 2.4.2 Graphique 2 – Évolution mensuelle des ventes de 2001 à 2005

**Objectif :** observer comment les ventes évoluent dans le temps.

Pour ce graphique, j'ai utilisé `store_sales` et `date_dim`.

J'ai relié les deux tables avec l'identifiant de date, puis regroupé les montants par mois.

J'ai obtenu **60 mois**, de janvier 2001 à décembre 2005.

J'ai utilisé une courbe pour représenter cette évolution.

**Résultats observés**

Sur la majorité de la période, les montants mensuels sont généralement autour de **18 à 20 millions**.

Le mois d'**octobre 2003** atteint **20 632 952,65**, ce qui représente le montant le plus élevé.

En dehors du dernier mois, février 2002 fait partie des mois avec les montants les plus faibles, avec **17 591 923,52**.

Le résultat qui attire surtout l'attention est celui de **décembre 2005**, avec seulement **8 078 218,76**.

**Analyse**

En observant la courbe, je remarque que les ventes varient selon les mois, mais qu'elles restent assez régulières sur la plupart des cinq années.

Je ne vois pas de progression constante ou de baisse régulière.

Par contre, la forte chute de décembre 2005 est différente des autres variations.

Au début, j'ai pensé que le dernier mois pouvait être incomplet. Mais je ne peux pas le confirmer avec les résultats obtenus.

Il faudrait vérifier les dates disponibles dans la table pour comprendre si cette baisse vient d'un manque de données ou d'une autre raison.

Ce graphique m'a permis de voir qu'une valeur inhabituelle ne doit pas être interprétée directement sans vérification.

**Source :** `store_sales` et `date_dim`, somme des montants par mois.

#### 2.4.3 Graphique 3 – Top 10 des produits selon le montant des ventes

**Objectif :** identifier les produits qui représentent les montants de ventes les plus importants.

J'ai regroupé les données `store_sales` par identifiant de produit, puis additionné les montants de vente.

J'ai également utilisé la table `item` pour consulter les informations sur les produits.

Comme les noms des produits n'étaient pas facilement lisibles, j'ai préféré utiliser leurs identifiants sur le graphique.

J'ai ensuite sélectionné les dix produits avec les montants les plus élevés.

**Résultats observés**

Les quatre premiers produits sont :

| Produit | Montant total |
|---|---:|
| 10000 | 15 408 393,55 |
| 10003 | 14 933 870,10 |
| 10001 | 14 572 124,66 |
| 10002 | 12 817 437,86 |

Le cinquième produit, **17502**, a un montant de **3 336 529,83**.

**Analyse**

Ce qui m'a surtout marquée, c'est la différence entre les quatre premiers produits et les autres.

Les quatre premiers sont nettement au-dessus du reste du classement.

En additionnant leurs montants, on obtient environ **57,73 millions**.

Cela montre que certains produits contribuent beaucoup plus au montant des ventes que d'autres.

Mais cela ne veut pas forcément dire que ces produits sont les plus vendus en quantité.

Un produit peut avoir un montant total important parce qu'il coûte plus cher.

Pour compléter l'analyse, il serait intéressant de comparer les quantités vendues et les prix des produits.

**Source :** `store_sales` et `item`, regroupement des ventes par identifiant de produit.

### 2.5 Bilan de l'analyse

Après avoir réalisé les calculs et les graphiques, j'ai retenu trois observations.

D'abord, les montants des ventes sont assez proches entre les 12 magasins, même si le magasin 7 arrive en première position. Il faudrait comparer d'autres indicateurs pour comprendre les différences.

Ensuite, quatre produits ressortent clairement du classement des ventes. Il serait utile de regarder leurs quantités et leurs prix avant de conclure qu'ils se vendent davantage.

Enfin, le graphique mensuel montre une forte chute en décembre 2005. Cette valeur mérite une vérification avant d'être interprétée.

Il faut aussi rappeler que les données TPCx-BB sont **synthétiques**. Ces résultats permettent de s'entraîner à l'analyse, mais ne représentent pas les ventes réelles d'une entreprise.

## 3. UC-07 – Architecture Azure

### 3.1 Création du stockage Azure

Pour la partie cloud, j'ai utilisé le parcours Azure avec mon compte étudiant.

Je suis allée sur le portail Azure et j'ai commencé par créer un groupe de ressources, puis un compte de stockage.

Pendant la configuration du compte de stockage, j'ai activé le **Hierarchical Namespace (HNS)** pour utiliser Azure Data Lake Storage Gen2.

Ensuite, dans la partie **Conteneurs**, j'ai créé un conteneur appelé :

`retail-intelligence`

Dans ce conteneur, j'ai organisé les dossiers de cette façon :

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

Cette organisation reprend le principe Bronze, Silver et Gold vu en cours.

Le dossier Bronze sert à conserver les données sources.

Silver sert à accueillir les données nettoyées ou préparées.

Gold sert aux résultats des calculs et aux indicateurs.

Les noms des dossiers sont simples pour pouvoir retrouver facilement les données. Dans un projet plus complet, il faudrait aussi prévoir des règles pour les dates de traitement, les versions et la conservation des fichiers.

### 3.2 Test du stockage dans Bronze

Pour vérifier que le stockage Azure fonctionnait, j'ai préparé un petit fichier de démonstration à partir de `store_sales.dat`.

Avec Python dans JupyterLab, j'ai récupéré les **100 premières lignes** du fichier.

J'ai enregistré cet échantillon sous le nom :

`echantillon_store_sales.dat`

Sa taille était de **13 394 octets**.

Ensuite, sur Azure, j'ai ouvert le conteneur `retail-intelligence`, puis le dossier `bronze/tpcxbb`.

J'ai utilisé l'option **Charger** pour envoyer le fichier.

Le chemin obtenu était :

`retail-intelligence/bronze/tpcxbb/echantillon_store_sales.dat`

Après le chargement, j'ai ouvert le fichier pour vérifier son contenu.

Les lignes séparées par le caractère `|` étaient visibles.

Azure affichait un avertissement concernant l'extension `.dat`, mais cela n'empêchait pas de consulter les données.

J'ai conservé des captures d'écran de cette manipulation.

### 3.3 Rôle du HNS

Le **Hierarchical Namespace** permet d'organiser les données avec de vrais dossiers et sous-dossiers.

Dans notre cas, il permet de retrouver facilement les zones Bronze, Silver et Gold.

Il facilite aussi les opérations sur les chemins et la gestion des droits d'accès aux répertoires.

Sans HNS, un stockage objet fonctionne principalement avec des objets et des préfixes dans leurs noms.

Avec HNS, on dispose d'une organisation hiérarchique plus adaptée au Data Lake.

### 3.4 Sécurité : RBAC et ACL

J'ai ensuite consulté la sécurité du compte de stockage dans Azure.

Dans **Contrôle d'accès (IAM)**, j'ai trouvé le rôle **Lecteur** associé à mon accès.

Ce rôle permet de consulter les ressources Azure, mais ne signifie pas automatiquement que tous les fichiers sont accessibles.

J'ai ensuite regardé les permissions ACL du dossier `bronze`.

| Identité | Lecture | Écriture | Exécution |
|---|---|---|---|
| Propriétaire | Oui | Oui | Oui |
| Groupe propriétaire | Oui | Non | Oui |
| Autres | Non | Non | Non |

Les permissions correspondent à `rwxr-x---`, soit `750`.

Cela signifie que le propriétaire peut lire, écrire et parcourir le dossier. Le groupe propriétaire peut le lire et le parcourir, mais pas écrire dedans. Les autres n'ont aucune permission ACL sur ce dossier.

J'ai aussi étudié la différence entre les mécanismes de sécurité :

- **Microsoft Entra ID** sert à gérer les identités et l'authentification.
- **Azure RBAC** sert à attribuer des rôles aux utilisateurs sur les ressources Azure.
- **ACL** sert à gérer plus précisément les permissions sur les dossiers et les fichiers.

Pour le projet, j'ai retenu cette proposition de droits :

| Profil | Bronze | Silver | Gold |
|---|---|---|---|
| Équipe ingestion | Écriture contrôlée | Lecture limitée | Aucun accès |
| Data Engineer | Lecture / écriture | Lecture / écriture | Écriture selon besoin |
| Analyste | Aucun accès par défaut | Lecture si nécessaire | Lecture |
| Administrateur | Selon les autorisations | Selon les autorisations | Selon les autorisations |

Cette matrice est une proposition. Je ne l'ai pas appliquée à de nouveaux utilisateurs dans Azure.

Le but est de respecter le **principe du moindre privilège**, c'est-à-dire donner uniquement les autorisations nécessaires à chaque utilisateur.

Il faut aussi protéger les mots de passe, les clés et les tokens d'accès et éviter de les enregistrer dans les fichiers publiés sur GitHub.

### 3.5 Azure Data Factory et Synapse

J'ai également étudié les autres services Azure proposés dans le cours.

**Azure Data Factory** sert à déplacer des données et à organiser l'exécution automatique des traitements.

**Azure Synapse Analytics** propose plusieurs outils pour analyser et traiter les données.

**Synapse Studio** est l'interface utilisée pour travailler avec ces services.

**Synapse Spark** permet d'effectuer des traitements distribués avec Spark.

**Synapse SQL** permet d'interroger des données avec des requêtes SQL.

Dans mon TP, je n'ai pas créé de pipeline Data Factory ni de workspace Synapse.

J'ai seulement étudié leur rôle, car le déploiement n'était pas obligatoire.

### 3.6 Architecture du projet

Le fonctionnement que j'ai réellement utilisé peut être résumé ainsi :

```text
              Données TPCx-BB
                    |
             JupyterLab / PySpark
                    |
             Préparation Silver
                    |
              Gold en Parquet
                    |
              KPI et graphiques
                    |
                Matplotlib
```

Les fichiers Silver et Gold ont été créés localement dans Docker.

Pour la partie Azure, j'ai testé séparément le stockage :

```text
              Données TPCx-BB
                    |
             Échantillon 100 lignes
                    |
                 Azure
                    |
               ADLS Gen2
                    |
             Bronze / tpcxbb
```

Les dossiers Silver et Gold existent sur Azure, mais ils n'ont pas été alimentés avec les fichiers Parquet du notebook.

Une chaîne automatique avec Data Factory et Synapse serait possible, mais elle n'a pas été réalisée pendant ce TP.

### 3.7 Comparaison Azure et open source

Dans le cours, nous avons aussi comparé les outils Azure avec leurs alternatives open source.

| Fonction | Azure | Alternative possible |
|---|---|---|
| Stockage des données | ADLS Gen2 | MinIO ou disque local |
| Traitement | Synapse Spark | Spark / PySpark |
| Tables analytiques | Tables du Lakehouse | Iceberg et Parquet |
| Requêtes SQL | Synapse SQL | Spark SQL ou Trino |
| Orchestration | Data Factory | Scripts ou Airflow |
| Métadonnées | Services Azure | Nessie ou Hive Metastore |
| Visualisation | Power BI | Matplotlib ou Superset |

La principale différence est qu'Azure propose plusieurs services gérés dans le cloud.

Avec les outils open source, il faut souvent installer et relier plusieurs composants.

Par exemple, Iceberg ne remplace pas à lui seul Synapse. Iceberg sert à gérer les tables et leurs métadonnées, tandis que Spark réalise les calculs et que Trino ou Spark SQL servent aux requêtes.

### 3.8 Limites et coûts Azure

Pour ce TP, j'ai limité l'utilisation réelle d'Azure au stockage ADLS Gen2 et au test d'un fichier.

Je n'ai pas lancé de traitements Spark sur Azure et je n'ai pas déployé Data Factory ou Synapse.

Ces services peuvent entraîner des coûts selon leur utilisation.

Même avec un compte étudiant, il faut surveiller les crédits disponibles, les ressources créées et les éventuelles opérations facturées.

Le choix de réaliser les traitements dans Docker permettait de travailler sur les données sans déployer toute une architecture cloud.

## 4. Difficultés rencontrées

J'ai rencontré plusieurs difficultés pendant le TP.

La première concernait les fichiers TPCx-BB. Les colonnes n'avaient pas de noms, donc j'ai dû examiner les schémas et les premières lignes pour comprendre les données.

Ensuite, PySpark a affiché un avertissement de compatibilité avec Pandas pendant certains traitements, mais les conversions utilisées ont fonctionné.

Lors de l'enregistrement des données Silver au format Parquet, Spark a aussi affiché un avertissement de mémoire (`MemoryManager`). Malgré cela, le fichier a été enregistré correctement.

Sur Azure, j'ai eu des difficultés à retrouver certains menus, notamment Configuration, Contrôle d'accès IAM et les permissions ACL.

J'ai donc réalisé les manipulations progressivement pour vérifier que chaque étape fonctionnait.

Enfin, pour publier le projet sur GitHub, j'ai utilisé Git depuis le terminal JupyterLab. J'ai dû configurer mon identité Git et régler l'authentification avant de réussir à envoyer les fichiers.

## Conclusion

Ce TP4 m'a permis de mieux comprendre les différentes étapes d'un projet Data.

Avec PySpark, j'ai chargé les données TPCx-BB, préparé les données Silver et Gold, calculé des indicateurs et réalisé trois graphiques.

L'analyse des résultats m'a permis de comparer les magasins, d'observer les ventes dans le temps et d'identifier les produits avec les montants les plus élevés.

J'ai aussi découvert une partie plus technique avec Docker, JupyterLab et Azure ADLS Gen2.

La manipulation Azure m'a permis de comprendre concrètement l'organisation Bronze, Silver et Gold et de tester le dépôt d'un fichier dans un Data Lake.

Tous les services Azure n'ont pas été déployés, mais j'ai pu comprendre leur rôle et la différence entre les traitements réalisés localement et ceux qui pourraient être automatisés dans le cloud.

Ce TP m'a surtout appris à vérifier les données avant de les analyser et à ne pas tirer de conclusions trop rapidement à partir d'un graphique.

Le fichier `tp4.ipynb` contient le code et les résultats, tandis que ce rapport présente les manipulations, les analyses et les difficultés rencontrées.
