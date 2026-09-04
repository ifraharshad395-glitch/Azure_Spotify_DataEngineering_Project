# Azure_Spotify_DataEngineering_Project

# Azure Spotify Data Engineering Project

## Project Overview

This project is an end-to-end **Data Engineering pipeline** built using Microsoft Azure and Databricks. The objective of the project is to ingest Spotify-related data from GitHub, process and transform the data through a **Medallion Architecture**, and create structured analytical tables in the Gold layer.

The pipeline follows a **Bronze → Silver → Gold** architecture, allowing raw data to be ingested first, cleaned and transformed in subsequent layers, and finally organized into dimensional and fact tables for analytics.

### Architecture

**GitHub → Azure Data Factory → Azure Data Lake Storage Gen2 → Databricks → Bronze → Silver → Gold**

The main Azure services and technologies used in this project include:

* Azure Resource Group
* Azure Data Lake Storage Gen2 (ADLS Gen2)
* Azure Data Factory
* Azure SQL Database
* Azure Databricks
* Azure Databricks Access Connector
* Unity Catalog / Metastore
* Apache Spark
* PySpark
* Auto Loader
* Spark Structured Streaming
* Delta Lake
* GitHub

![image alt](https://github.com/ifraharshad395-glitch/Azure_Spotify_DataEngineering_Project/blob/7e9eac9cb99dfa64175a4ccd5eb45646e35c6de4/incremental_pipeline.png)
---

## Project Architecture

The project uses the **Medallion Architecture**, with data progressing through three layers:

### Bronze Layer — Raw Data

The Bronze layer acts as the landing zone for the source data.

Spotify datasets are hosted on GitHub and retrieved using **Azure Data Factory**. The raw files are then stored in the Bronze container in ADLS Gen2 with minimal or no transformation.

This layer preserves the source data and provides a reliable starting point for downstream processing. 

![imgage alt](

### Silver Layer — Cleaned & Transformed Data

The Silver layer contains cleaned, standardized, and transformed data.

Azure Databricks and **PySpark** are used to process the Bronze data. **Auto Loader** and **Spark Structured Streaming** are used to incrementally detect and process incoming files.

Several transformations are performed in this layer, including data cleaning, schema handling, and preparing the datasets for analytical use.

The resulting data is stored in the Silver container using **Delta format**.

### Gold Layer — Analytical Data

The Gold layer contains business-ready datasets designed for analytics.

The transformed Silver data is further processed in Databricks and organized into a dimensional model consisting of:

* **Dim User**
* **Dim Artist**
* **Dim Track**
* **Fact Stream**

Delta tables are used in the Gold layer to support efficient storage and incremental updates.

---

## Azure Resource Setup

The project was organized inside an **Azure Resource Group** containing the required data engineering resources.

### Azure Data Lake Storage Gen2

An ADLS Gen2 storage account was created as the central data storage layer.

Three containers were created to represent the Medallion Architecture:

```text
ADLS Gen2
│
├── bronze
│
├── silver
│
└── gold
```

Each container corresponds to a different stage of the data pipeline.

---

## Data Ingestion with Azure Data Factory

**Azure Data Factory (ADF)** was used to build the ingestion pipeline responsible for retrieving source data from GitHub and loading it into the Bronze layer.

### Linked Services

Linked services were configured to establish connections between Azure Data Factory and:

* GitHub
* Azure Data Lake Storage Gen2

### Incremental Data Ingestion

The pipeline was designed to support **incremental ingestion**, allowing new or changed files to be processed without unnecessarily reprocessing the entire dataset.

ADF activities used in the pipeline include:

* **Lookup Activity** — used to retrieve and work with metadata about the source files.
* **Copy Data Activity** — used to transfer data from GitHub into ADLS Gen2.
* **If Condition Activity** — used to apply conditional logic within the pipeline.
* **Dynamic Content / Expression Builder** — used to dynamically reference files and parameters.
* **ForEach Activity** — used to iterate through multiple source files and process them individually.

The ingestion workflow dynamically identifies the files available in the source and loads them into the Bronze container.

---

## Azure Databricks

Azure Databricks was used as the main processing and transformation environment.

An **Azure Databricks workspace** was created along with an **Access Connector for Azure Databricks** to facilitate secure access to Azure resources.

A **Unity Catalog metastore** was also configured and the Databricks workspace was assigned to the metastore for centralized data governance and management.

---

## Silver Layer Processing

The Silver layer was implemented using **PySpark**, **Auto Loader**, and **Spark Structured Streaming**.

### Auto Loader

Databricks Auto Loader was used to incrementally detect newly arriving files in the Bronze storage layer.

Instead of repeatedly scanning and processing the entire directory, Auto Loader allows the pipeline to identify new files as they arrive and process them incrementally.

### PySpark Transformations

The Bronze datasets were loaded into Databricks using PySpark and their corresponding storage paths.

The data was then transformed to prepare it for the Silver layer. This included operations such as:

* Reading data from ADLS Gen2
* Applying schemas
* Cleaning and standardizing data
* Handling columns and data types
* Transforming datasets into structured tables
* Writing processed data in Delta format

The processed datasets were then stored in the Silver container.

---

## Gold Layer & Dimensional Model

The final stage of the pipeline transforms the Silver datasets into analytical tables in the Gold layer.

The Gold layer follows a **dimensional modeling approach**, separating descriptive entities into dimension tables and transactional/event information into a fact table.

### Dimension Tables

| Table          | Description                                                         |
| -------------- | ------------------------------------------------------------------- |
| **Dim User**   | Contains descriptive information about Spotify users.               |
| **Dim Artist** | Contains information about artists associated with streamed tracks. |
| **Dim Track**  | Contains descriptive information about the tracks being streamed.   |

### Fact Table

| Table           | Description                                                                                      |
| --------------- | ------------------------------------------------------------------------------------------------ |
| **Fact Stream** | Contains streaming events and connects users, artists, and tracks through their respective keys. |

Conceptually, the model can be represented as:

```text
                ┌──────────────┐
                │   Dim User   │
                └──────┬───────┘
                       │
                       │
                       ▼
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│  Dim Artist  │──►│  Fact Stream │◄──│  Dim Track   │
└──────────────┘   └──────────────┘   └──────────────┘
```

This structure makes the data easier to query and analyze while separating descriptive attributes from streaming events.

---

## Delta Lake

Delta Lake was used throughout the Silver and Gold layers to provide reliable, transactional storage on top of the data lake.

The Gold tables were implemented as **Delta tables**, allowing the pipeline to perform incremental updates rather than rebuilding the entire dataset whenever new data becomes available.

This also provides benefits such as:

* ACID transactions
* Schema enforcement
* Reliable incremental processing
* Efficient updates and merges
* Versioned data through Delta Lake

---

## End-to-End Data Flow

The complete pipeline can be summarized as:

```text
                    GitHub
                       │
                       ▼
             Azure Data Factory
                       │
            ┌──────────┴──────────┐
            │ Lookup / Copy /     │
            │ ForEach / If        │
            │ Dynamic Expressions │
            └──────────┬──────────┘
                       │
                       ▼
              ADLS Gen2 - Bronze
                       │
                       │ Auto Loader
                       │ + Spark Streaming
                       ▼
                Databricks
                       │
                       │ PySpark
                       │ Transformations
                       ▼
              ADLS Gen2 - Silver
                       │
                       │ Transformations
                       │ + Incremental Updates
                       ▼
              ADLS Gen2 - Gold
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
      Dim User    Dim Artist    Dim Track
          │            │            │
          └────────────┼────────────┘
                       ▼
                  Fact Stream
```

---

## Key Concepts Demonstrated

Through this project, I worked with several core Data Engineering concepts:

* Designing an end-to-end cloud data pipeline
* Medallion Architecture
* Data ingestion using Azure Data Factory
* Incremental data loading
* Dynamic pipelines using expressions
* Parameterized file processing
* Azure Data Lake Storage Gen2
* Azure Databricks
* PySpark
* Apache Spark
* Auto Loader
* Spark Structured Streaming
* Delta Lake
* Delta tables
* Data transformation and cleaning
* Dimensional modeling
* Fact and dimension tables
* Unity Catalog / Metastore
* Azure resource and storage management

---

## Technologies Used

| Category                          | Technology                              |
| --------------------------------- | --------------------------------------- |
| Cloud Platform                    | Microsoft Azure                         |
| Data Ingestion                    | Azure Data Factory                      |
| Data Lake                         | Azure Data Lake Storage Gen2            |
| Data Processing                   | Azure Databricks                        |
| Programming                       | Python / PySpark                        |
| Processing Engine                 | Apache Spark                            |
| Streaming / Incremental Ingestion | Auto Loader, Spark Structured Streaming |
| Storage Format                    | Delta Lake                              |
| Data Modeling                     | Dimensional Modeling                    |
| Source                            | GitHub                                  |
| Database                          | Azure SQL Database                      |
| Governance                        | Unity Catalog / Metastore               |
| Version Control                   | Git / GitHub                            |

---

## Project Outcome

The completed pipeline demonstrates how raw source data can be ingested from an external source, stored in a cloud data lake, incrementally processed using Spark, transformed through multiple data layers, and ultimately converted into structured analytical tables.

The project provided practical experience with **Azure Data Factory, ADLS Gen2, Databricks, PySpark, Auto Loader, Spark Structured Streaming, Delta Lake, and dimensional data modeling**, while demonstrating how these technologies can work together to build a modern cloud-based data engineering pipeline.


