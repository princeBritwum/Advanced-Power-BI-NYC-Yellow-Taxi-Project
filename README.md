# Advanced-Power-BI-US-domestic-flights-Project
In this Project I use advanced features in Power BI such as Incremental refresh using XMLA End point, Row Level Security and Tabular Editor to Make Metadata Changes and Incremental changes

## Project Overview

This project demonstrates a retail data pipeline built on Azure. The pipeline ingests raw sales data from an on-premise server, processes it through ETL steps, and transforms it for analytics and reporting using Azure services such as Data Factory, Azure Synapse Analytics, Databricks, and SQL Data Warehouse. 

## Architecture

![docs/Data Architecture.png](https://github.com/princeBritwum/Azure-Retail-Data-Engineering-Project/blob/main/docs/Data%20Architecture.png)

The architecture includes:
- **Self-Hosted Integration runtimes** for loading data from on-prem datasource
- **Azure Data Factory / Azure Synapse Analytics** for orchestrating data movement.
- **Azure Databricks** for data transformation and processing.
- **Azure Data Lake** for scalable data storage.
- **Azure Dedicated SQL Pool** for data warehousing and reporting.

## Features
- **Data Ingestion:** Automated pipelines for ingesting raw sales data.
- **ETL Processing:** Transformation of raw data into clean, analytics-ready datasets.
- **Data Analytics:** Insights and reporting using Power BI.

## Getting Started

### Prerequisites

- Python 3.8+
- Azure Subscription
  1. Synapse Workspace
  2. Dedicated SQL Pool
  3. Data Lake Gen 2
  4. Databricks
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
- Jupyter Notebook
- On-Premise Server with MSSQL Db

### Setup
