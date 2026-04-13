# Projet-Analyse-des-Offres-d-Emploi-LinkedIn-avec-Snowflake
Analyse de bout en bout du marché de l’emploi LinkedIn à l’aide de Snowflake (architecture Bronze/Silver/Gold) et d’un tableau de bord interactif Streamlit.
## 1. Introduction 

Ce projet a pour objectif d’analyser le marché de l’emploi LinkedIn à partir de données hétérogènes (CSV et JSON) stockées dans un bucket S3 public. Il s’inscrit dans une démarche complète de Data Engineering, allant de l’ingestion brute des données jusqu’à leur visualisation à travers un dashboard interactif. 

L’enjeu principal est double : 

Mettre en place un pipeline de données robuste et traçable dans Snowflake, basé sur l’architecture Bronze / Silver / Gold. 

Fournir des analyses exploitables via SQL et Streamlit afin de mieux comprendre les dynamiques du marché de l’emploi :
types de postes, niveaux de salaire, secteurs d’activité, tailles d’entreprises et compétences recherchées. 

Ce rapport détaille pas à pas chaque étape, explique chaque script SQL, justifie les choix techniques, et revient en profondeur sur les problèmes rencontrés et leurs solutions. 
## 2. Étapes Réalisées
### 2. 1. Création de la Base de Données

Le script commence par la création d’une base de données nommée LINKEDIN, suivie du schéma BRONZE. Cette étape est fondamentale, car elle initialise l’espace de travail dans lequel toutes les données brutes seront déposées.
L’utilisation de IF NOT EXISTS garantit que la création est idempotente : le script peut être relancé plusieurs fois sans créer de doublons ou générer d’erreurs

```sql
-- Create Databse
CREATE  DATABASE IF NOT EXISTS  linkedin;

```
### 2.2. Création du schéma Bronze 
text exp


```sql
-- Create Schema BRONZE
CREATE SCHEMA IF NOT EXISTS linkedin.BRONZE;

```
### 2.3. Configuration du Stage Externe

un stage Snowflake est configuré pour pointer vers un bucket S3 public. Ce stage joue le rôle d’un connecteur externe permettant à Snowflake d’accéder directement aux fichiers CSV et JSON stockés dans le cloud. Cette étape prépare donc l’ingestion des données provenant de LinkedIn.
```sql
-- Create Stage  
CREATE OR REPLACE STAGE LINKEDIN.BRONZE.linkedin_stage
URL = 's3://snowflake-lab-bucket/';

```
### 2.4 Création des tables et chargement des données
Pour chaque type de fichier (job postings, benefits, skills, employee counts…), une table est créée dans la couche BRONZE avec toutes les colonnes en STRING. Ce choix volontaire suit la philosophie de la couche BRONZE : stocker la donnée telle qu’elle existe, sans transformation, sans typage, sans prise de décision métier. Cela garantit une ingestion fiable, même si les fichiers contiennent des irrégularités.


La commande COPY INTO est ensuite utilisée pour importer les données depuis le stage S3 vers Snowflake. L’option SKIP_HEADER=1 permet d’éviter l’ingestion de la ligne d’en‑tête des CSV, tandis que l’option FIELD_OPTIONALLY_ENCLOSED_BY sécurise l’ingestion des champs contenant des guillemets. Après chaque chargement, une requête SELECT * assure une vérification instantanée du contenu de la table BRONZE. 


Les fichiers JSON sont eux aussi ingérés dans des tables BRONZE, mais contrairement aux CSV, ils sont stockés dans une unique colonne VARIANT. Cela permet de conserver la structure JSON originale, avec ses attributs imbriqués. Une conséquence directe est que chaque fichier JSON contenant un tableau est ingéré sous forme d’une seule ligne, ce qui nécessitera une correction en SILVER.


*  Table Benefits :


 
