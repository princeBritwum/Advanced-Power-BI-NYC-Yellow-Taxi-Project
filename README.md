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

2. First of we need to convert our parquet file to csv and store it back in the file server with the code below;
   ```ipynb
        import pyarrow.parquet as pq
        import pandas as pd
        df = pd.read_parquet(r"C:\Users\PrinceBritwum\Documents\yellow_tripdata_2016-03.parquet")
        df.to_csv(r"C:\Users\PrinceBritwum\Documents\yellow_tripdata_2016-03.csv")
   
2. Next , we need to create our staging table where we will load the transaction files from the file store to:
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
   
3. We use SQL Bulk Insert function to insert transactions into the staging table
   ```sql
        BULK INSERT [dbo].[StageNycTrip]
        FROM 'C:\Users\PrinceBritwum\Documents\yellow_tripdata_2016-03.csv'
        WITH (FIRSTROW = 2, 
        	  FIELDTERMINATOR = ',', 
        	  ROWTERMINATOR = '\n');
        
        SELECT TOP (1000)* FROM StageNycTrip;
4. Now we need the landing table which will be the source table for our Power BI report. As we mentioned earlier, we are dealing with quite a large number of rows with about 12 million records per month. To ensure we get our table querty optimized for performance and speed, we will do a number of things;
- Create our table with partitions to to store each months' data into a partition
- Next we will create a clustered and non clustered index on the columns the partition keys and the date columns respectively.
- Finally crate the table with the partitioned defined

  ```sql
      
    CREATE PARTITION FUNCTION NycTripFunc (int)  
        AS RANGE RIGHT FOR VALUES ( 20160301,20160401,20160501,20160601,20160701,20160801,20160901,20161001,20161101,20161201,20170101 ) ;
      
    --b. Create the partition schema using the partition function
    
    CREATE PARTITION SCHEME NycTripSchema  
        AS PARTITION NycTripFunc  
        ALL TO ('PRIMARY') ;  
    
    --4. Create the NycTrip Table 
    
    CREATE SEQUENCE NycSeq
    START WITH 1
    INCREMENT BY 1;

  
  CREATE TABLE [dbo].[PartNycTrip](
  	
      [TripID] INT DEFAULT NEXT VALUE FOR NycSeq,
  	[LoadDate] [date] NULL,
  	[pickupdatekey] [int] NOT NULL,
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
      CONSTRAINT [PKNycTrip_TripPickupdatekey] PRIMARY KEY CLUSTERED 
  (	
  	[TripID] ASC,
  	[pickupdatekey] ASC
  
  ))
  ON NycTripSchema ([pickupdatekey]);

    --CREATE NON CLUSTERED INDEXES
    
    CREATE NONCLUSTERED INDEX PKNycTrip_PickDropDate
    ON [dbo].[PartNycTrip]([tpep_pickup_datetime],[tpep_dropoff_datetime]);
    
    CREATE NONCLUSTERED INDEX PKNycTrip_LoadDate
    ON [dbo].[PartNycTrip]([LoadDate]);

5. Now we have all our tables set up, we will now begin to insert data from the staging table into the landing table and check our partitions and the data loaded into it
	  ```sql
		 
		INSERT INTO [PartNycTrip] ([TripID], [LoadDate] , [pickupdatekey] , [VendorID] , [tpep_pickup_datetime] , [tpep_dropoff_datetime] , [passenger_count] , [trip_distance] , 		[RatecodeID] , [store_and_fwd_flag] , [PULocationID] , [DOLocationID] , [payment_type] , [fare_amount] , [extra] , [mta_tax] , [tip_amount] , [tolls_amount] ,
		[improvement_surcharge] , [total_amount] , [congestion_surcharge] , [airport_fee])
		SELECT
		 NEXT VALUE FOR NycSeq AS [TripID],
		 CAST(GETDATE() AS DATE) AS [LoadDate],
		 FORMAT([tpep_pickup_datetime], 'yyyyMMdd') AS [pickupdatekey] ,
		 [VendorID] ,
		 [tpep_pickup_datetime],
		 [tpep_dropoff_datetime] ,
		 [passenger_count] ,
		 [trip_distance] ,
		 [RatecodeID] ,
		 [store_and_fwd_flag] ,
		 [PULocationID] ,
		 [DOLocationID] ,
		 [payment_type] ,
		 [fare_amount] ,
		 [extra] ,
		 [mta_tax] ,
		 [tip_amount] ,
		 [tolls_amount] ,
		 [improvement_surcharge] ,
		 [total_amount] ,
		 [congestion_surcharge] ,
		 [airport_fee]
		FROM [AWDW2022].[dbo].[StageNycTrip]
		WHERE FORMAT([tpep_pickup_datetime], 'yyyyMMdd') > FORMAT(DATEADD(MONTH, 0, CAST(STUFF(STUFF(CONVERT(VARCHAR(8), FORMAT([tpep_pickup_datetime], 'yyyyMMdd')), 5, 0, '-'), 8, 		0, '-') 	AS DATE)), 'yyyyMMdd')
		  AND FORMAT([tpep_pickup_datetime], 'yyyyMMdd') <= FORMAT(DATEADD(MONTH, 1, CAST(STUFF(STUFF(CONVERT(VARCHAR(8), FORMAT([tpep_pickup_datetime], 'yyyyMMdd')), 5, 0, '-'), 		8, 0, '-
		') AS DATE)), 'yyyyMMdd')
   
	--Lets check the partitions with the query
   
	SELECT SCHEMA_NAME(t.schema_id) AS SchemaName, t.name AS TableName, i.name AS IndexName, 
	    p.partition_number AS PartitionNumber, f.name AS PartitionFunctionName, p.rows AS Rows, rv.value AS BoundaryValue, 
	CASE WHEN ISNULL(rv.value, rv2.value) IS NULL THEN 'N/A' 
	ELSE
	    CASE WHEN f.boundary_value_on_right = 0 AND rv2.value IS NULL THEN '>=' 
	        WHEN f.boundary_value_on_right = 0 THEN '>' 
	        ELSE '>=' 
	    END + ' ' + ISNULL(CONVERT(varchar(64), rv2.value), 'Min Value') + ' ' + 
	        CASE f.boundary_value_on_right WHEN 1 THEN 'and <' 
	                ELSE 'and <=' END 
	        + ' ' + ISNULL(CONVERT(varchar(64), rv.value), 'Max Value') 
	END AS TextComparison
	FROM sys.tables AS t  
	JOIN sys.indexes AS i  
	    ON t.object_id = i.object_id  
	JOIN sys.partitions AS p  
	    ON i.object_id = p.object_id AND i.index_id = p.index_id   
	JOIN  sys.partition_schemes AS s   
	    ON i.data_space_id = s.data_space_id  
	JOIN sys.partition_functions AS f   
	    ON s.function_id = f.function_id  
	LEFT JOIN sys.partition_range_values AS r   
	    ON f.function_id = r.function_id and r.boundary_id = p.partition_number  
	LEFT JOIN sys.partition_range_values AS rv
	    ON f.function_id = rv.function_id
	    AND p.partition_number = rv.boundary_id     
	LEFT JOIN sys.partition_range_values AS rv2
	    ON f.function_id = rv2.function_id
	    AND p.partition_number - 1= rv2.boundary_id
	WHERE 
	    t.name = 'PartNycTrip'
	    AND i.type <= 1 
	ORDER BY t.name, p.partition_number;


![docs/Partition.png](https://github.com/princeBritwum/Advanced-Power-BI-NYC-Yellow-Taxi-Project/blob/main/docs/Partition.png)

6. Our data is correctly loaded into the partittions we created as shown above, we are now ready to connect our landing table to Power BI.

![docs/Landing Table.png](https://github.com/princeBritwum/Advanced-Power-BI-NYC-Yellow-Taxi-Project/blob/main/docs/Landing%20Table.png)

**We have our database ready, connect Power BI to it and delve into that**

1. To emplement incremental refresh on our landing table, we need to ensure the following;
- Our table is able to query fold - To make this possible, ensure you have done most of the basic transformation on the MSSQL Query and create a materialized view for the table. In this case I created a view called dbo_PartNyCTripView and used it as my source in the query editor as shown below in the mquery
- Create rangeStart and rangeEnd parameters with dates as specified below
- filter your incremental date column in our case (LoadDate) on table(NycTripW45) with the parameters defined in the previous step


![docs/QueryEditor.png](https://github.com/princeBritwum/Advanced-Power-BI-NYC-Yellow-Taxi-Project/blob/main/docs/QueryEditor.png)

This is the mquery for the table (NycTripW45) in query editor

	
 	let
    Source = Sql.Database("WORKSTATION-006", "AWDW2022", [CommandTimeout=#duration(0, 2, 0, 0)]),
    dbo_PartNyCTripView = Source{[Schema="dbo",Item="PartNyCTripView"]}[Data],
    #"Changed Type" = Table.TransformColumnTypes(dbo_PartNyCTripView,{{"LoadDate", type datetime}}),
    #"Filtered Rows" = Table.SelectRows(#"Changed Type", each [LoadDate] >= RangeStart and [LoadDate] < RangeEnd)
	in
    #"Filtered Rows"

Now we can set up our incremental refresh on the table (NycTripW45) below;

![docs/incremental_refresh.png](https://github.com/princeBritwum/Advanced-Power-BI-NYC-Yellow-Taxi-Project/blob/main/docs/incremental_refresh.png)


2. Save the Power Bi report when the incremental refresh is enabled and publish to the premium capacity workspace. To work with incremental refresh and make meta data changes to your data model, you need to enable xmla endpoints in the Admin portal under Governance and insight in the settings page.


![docs/xmla.png](https://github.com/princeBritwum/Advanced-Power-BI-NYC-Yellow-Taxi-Project/blob/main/docs/xmla.png)

   
4. Once the report is published, go ahead to the service and setup your gateway and datasource connections. Please note that when you pusblish a dataset with incremental refresh defined, partitions are created according what was defined in the Power BI desktop. In our scenario, data will be archived for the past year and new data will be loaded every day into the dataset when the maximum record of the column(LoadDate) changes ie if the max record of LoadDate detected in our view [dbo_PartNyCTripView] is higher than the present record in the data(semantic) model. This partition however is not created until you refresh power BI on the service for the first time. To check the partition created by incremental refresh, you can use the Analysis services in SQL management studio or Tabular Editor.
   - To connect to Analysis services, you will need the connection string in the setting of the semantic model of your published report.
	![docs/connectionString.png](https://github.com/princeBritwum/Advanced-Power-BI-NYC-Yellow-Taxi-Project/blob/main/docs/connectionString.png)

   - Next connect to the Analysis Services and authenticate with your Microsoft Entra details
     ![docs/AnalysisServices.png](https://github.com/princeBritwum/Advanced-Power-BI-NYC-Yellow-Taxi-Project/blob/main/docs/AnalysisServices.png)

