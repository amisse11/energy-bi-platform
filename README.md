# Energy BI Platform - Suivi de la consommation énergétique

Plateforme de données complète pour l'analyse de la consommation électrique d'un foyer.
Couvre le cycle complet de l'ingestion à la visualisation : ADF > Databricks > Azure SQL > Power BI.

***

## Stack technique

- Azure Data Factory - orchestration du pipeline
- Azure Databricks - transformation et modélisation (PySpark / Python)
- Azure Blob Storage - couche de stockage Medallion (Bronze / Silver / Gold)
- Azure SQL Database - entrepôt de données (Star Schema)
- Azure Key Vault - gestion des secrets et credentials
- Power BI Desktop - dashboard analytique (3 pages)
- Python / SQL - transformations et requêtes

***

## Dataset

Source : UCI Household Electric Power Consumption Dataset

Localisation : Sceaux, France - 7 km au sud de Paris

Période : Décembre 2006 à Novembre 2010 (47 mois)

Volume : 2 075 259 mesures à la minute

Sous-compteurs : Cuisine, Buanderie, Chauffe-eau / Climatisation

***

## Architecture du pipeline

```
CSV Source
    |
Azure Data Factory
  - Validation de la source (Get Metadata)
  - Copie vers Bronze (Copy Data)
  - Déclenchement Silver (Databricks Notebook 02)
  - Déclenchement Star Schema (Databricks Notebook 04)
    |
Azure Blob Storage (Bronze > Silver > Gold)
    |
Azure Databricks
  - 01 : Ingestion Bronze
  - 02 : Nettoyage Silver (PySpark)
  - 03 : Agrégation Gold
  - 04 : Construction Star Schema > Azure SQL
    |
Azure SQL Database (Star Schema)
    |
Power BI Dashboard (3 pages analytiques)
```

***

## Modèle dimensionnel (Star Schema)

Quatre tables dans Azure SQL :

**DimDate** (1 442 lignes)
- Attributs temporels enrichis : année, mois, trimestre, jour de semaine, indicateur weekend

**DimMeter** (1 ligne)
- Informations du compteur : nom, type, localisation, fréquence d'échantillonnage

**FactEnergyGlobal** (1 442 lignes)
- Consommation journalière globale : puissance active, tension moyenne, intensité, énergie non mesurée

**FactSubMetering** (4 326 lignes)
- Consommation journalière par sous-compteur (Cuisine, Buanderie, Chauffe-eau / Clim)

***

## Notebooks Databricks (production)

- `01_bronze_ingestion.ipynb` - Lecture des données brutes depuis Azure Blob
- `02_silver_transform.ipynb` - Nettoyage, typage, suppression des valeurs manquantes (PySpark)
- `03_gold_aggregation.ipynb` - Agrégation journalière vers la couche Gold
- `04_gold_star_schema.ipynb` - Construction du Star Schema et chargement JDBC vers Azure SQL

## Notebooks locaux (prototypage R&D)

Ces notebooks représentent la phase d'exploration et de conception qui a précédé l'industrialisation sur Azure.

- `01_exploration_uci_power.ipynb` - Exploration et analyse descriptive du dataset UCI
- `02_preparation.ipynb` - Nettoyage initial et préparation des données (pandas)
- `03_schema_entrepot.ipynb` - Conception du modèle dimensionnel (Star Schema)
- `04_chargement_azure_sql.ipynb` - Prototype de chargement vers Azure SQL

***

## Dashboard Power BI

**Page 1 - Vue d'ensemble**
- KPI : Consommation totale, Moyenne journalière, % Énergie non mesurée
- Area Chart : Évolution de la consommation mensuelle sur 4 ans
- Clustered Bar : Consommation Weekend vs Semaine par année
- Slicer : Filtre par année

**Page 2 - Analyse par zone**
- Stacked Bar : Consommation mensuelle par sous-compteur
- Matrix : Consommation par zone et trimestre (formatage conditionnel)
- Donut : Répartition de la consommation par zone
- Line : Tension moyenne vs Intensité mensuelle
- Slicers : Année et Trimestre

**Page 3 - Time Intelligence**
- KPI : Consommation YTD, Variation MoM %
- Area Chart : Progression YTD - 4 années superposées (2007–2010)
- Line Chart : Variation mensuelle (%) avec ligne de référence à 0%

***

## Résultats

- 2 075 259 mesures brutes traitées
- 1 442 jours couverts (47 mois)
- 4 tables SQL chargées automatiquement
- Pipeline entièrement automatisé (0 intervention manuelle)
- 9 mesures DAX avancées (YTD, MoM, Weekend vs Semaine)

***

## Structure du repo

```
energy-bi-platform/
│   .env
│   .env.example
│   .gitignore
│   publish_config.json
│   README.md
│   
├───data
│   ├───interim
│   ├───processed
│   │       daily_power_consumption.csv
│   │       daily_power_consumption.parquet
│   │       hourly_power_consumption.csv
│   │       hourly_power_consumption.parquet
│   │       
│   ├───raw
│   │       household_power_consumption.txt
│   │       
│   └───schema
│           DimDate.csv
│           DimMeter.csv
│           FactConsumption.csv
│           
├───dataset
│       ds_blob_bronze.json
│       ds_csv_raw.json
│       
├───docs
│   │   bi-energie.pbix
│   │   
│   └───screenshots
│           01_powerbi_overview.png
│           02_powerbi_zones.png
│           03_powerbi_time_intelligence.png
│           04_powerbi_model_view.png
│           05_adf_pipeline.png
│           06_adf_debug_success.png
│           07_azure_sql_rowcount.png
│           
├───factory
│       energy-bi-adf.json
│       
├───linkedService
│       AzureSqlDatabase1.json
│       ls_azure_sql.json
│       ls_blob_raw.json
│       ls_databricks_energy.json
│       
├───notebooks
│   ├───databricks
│   │       01_bronze_ingestion.ipynb
│   │       02_silver_transform.ipynb
│   │       03_gold_aggregation.ipynb
│   │       04_gold_star_schema.ipynb
│   │       
│   └───local
│           01_exploration_uci_power.ipynb
│           02_preparation_uci_power.ipynb
│           03_schema_entrepot.ipynb
│           04_chargement_azure_sql.ipynb
│           
├───pipeline
│       pl_ingest_energy_csv.json
│       
└───src
        __init__.py
```

***

## Auteur

Anthony Misse