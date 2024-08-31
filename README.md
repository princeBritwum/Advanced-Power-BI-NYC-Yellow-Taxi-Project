# Advanced Power BI Yellow Taxi Trip Project
In this Project I use advanced features in Power BI such as Incremental refresh using XMLA End point, Row Level Security and Tabular Editor to Make Metadata Changes and Incremental changes

## Project Overview

This project demonstrates a retail data pipeline built on Azure. The pipeline ingests raw sales data from an on-premise server, processes it through ETL steps, and transforms it for analytics and reporting using Azure services such as Data Factory, Azure Synapse Analytics, Databricks, and SQL Data Warehouse. 

## Architecture

![docs/architecture_powerBI.png](https://github.com/princeBritwum/Advanced-Power-BI-NYC-Yellow-Taxi-Project/blob/main/docs/architecture_powerBI.png)

The architecture includes:
- **File Store** for storing transactional data in parquet
- **Ipynb Notebook and MSSQL Staging Table** for extracting parquet data to CSV for bulk insert load.
- **MSSQL Landing Table** for data Data Storage.
- **Power BI Desktop** for Report development.
- **Power BI Service** for managing published reports.

## Features
- **Data Ingestion:** Automated pipelines for ingesting raw sales data.
- **ETL Processing:** Transformation of raw data into clean, analytics-ready datasets.
- **Data Analytics:** Insights and reporting using Power BI.

## Getting Started

### Prerequisites

- Python 3.8+
- Power BI
  1. Power BI Desktop
  2. Power BI Service with Premium Capacity
  3. Power BI Pro-License
  4. Databricks
- MSSQL Database
  1. Stating Table
  2. Landing Table


### Problem Statement
The client have business transactions exported into parquet files and stored in a secure file server location every night. The client transaction system generates well over 12 million records every month. He wants to utilize the Power BI for analytics but then also have a MSSQL datastore where all these transaction files will be transformed and securely stored for future use.


### Approach
**As indicated in the architecture, this is how we plan to implement the solution**
- First of we use pandas to transform the transaction files in parquet to csv
- When we have the files in CSV , we will then use Bulk insert to data from fileserver to a staging table
- We will then insert into the lading table which has have been properly partitioned for performance when using as a source for Power BI and other anlytics activities
- In Power BI we will implement incremental refresh to incrementaly load data from landing table (source table) into Power BI.
- We will also define roles level security in the report to provide secured access and view to only allowed and permited users
- We will make use of XMLA endpoint to make meta data changes to our report since we dont want to break the incremental partions created when we published to Power BI Service
- We would also use Tabular Editor 3 to see how to manage the partitions created by incremental refresh

**We have a lot to do so lets dive in >>**



### Setup
1. Clone this repository:
   ```bash
   git clone https://github.com/princeBritwum/Advanced-Power-BI-NYC-Yellow-Taxi-Project.git
   cd Advanced-Power-BI-NYC-Yellow-Taxi-Project

2. First off, we need to create our staging table:
   ```sql
       CREATE TABLE [dbo].[StageNycTrip](
      	[VendorId] [int] NULL,
      	[tpep_pickup_datetime] [datetime] NULL,
      	[tpep_dropoff_datetime] [datetime] NULL,
      	[passenger_count] [int] NULL,
      	[trip_distance] [float] NULL,
      	[PULocationID] [float] NULL,
      	[DOLocationID] [float] NULL,
      	[RatecodeID] [int] NULL,
      	[store_and_fwd_flag] [nchar](10) NULL,
      	[payment_type] [int] NULL,
      	[fare_amount] [float] NULL,
      	[extra] [float] NULL,
      	[mta_tax] [float] NULL,
      	[Improvement_surcharge] [float] NULL,
      	[tip_amount] [float] NULL,
      	[tolls_amount] [float] NULL,
      	[total_amount] [float] NULL,
      	[Congestion_Surcharge] [float] NULL,
      	[Airport_fee] [float] NULL
      ) ON [PRIMARY]
   
