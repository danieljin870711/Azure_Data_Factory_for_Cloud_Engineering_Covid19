# Azure Data Factory for Cloud Engineering

As we approach a modern data warehouse over an on-prem server, it is critical to understand how to implement a robust cloud data warehouse. One of the key areas within cloud warehousing is to have proper data pipelines. This is how the data can be extracted, transformed, and loaded into the data warehouse. In this project, I will demonstrate how to build data pipelines with Azure Data Factory (ADF) for data ingestion, data transformation, data load, and data orchestration for advanced business intelligence analytics. 

## Project Covers... 
- Project Overview
- Azure Environment Setup
- Data Ingestion from Azure Blob
- Data Ingestion from HTTP
- Data Flows
- Copy Data to Azure SQL DB
- Data Orchestration

### Project Overview 

![](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/SolutionArchitect.png)
The purpose of this project is to demonstrate how to build data pipelines end-to-end from various data sources to Azure SQL DB for BI analytics. I used two data sources, one for ECDC Covid data using HTTP and another for Population data using Azure Blob Storage. Then, I used ADF for ingesting the data into Azure Data Lake Gen2. To perform data transformation, data flows within ADF are used. After the transformation, the transformed data is loaded into Azure SQL db. 



### Azure Environment Setup

**Account Creation**

![App Screenshot](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/Azure%20Account.png)
![](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/Subscription.png)

I created a new Azure account with a $200 credit to proceed with this project with the following subscription. After the creation of the account, then I installed the Azure Data Factory, Azure Storage Account, Azure Data Lake Storage Gen2, and Azure SQL DB. Also, I downloaded Azure Storage Explore which makes it easier to manage the Azure Storage within this app. 

Azure Data Factory
![](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/ADF.png)

Azure Storage Account
![](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/StorageAccount.png)

Azure SQL DB
![](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/SQLDB.png)

### Data Ingestion from Azure Blob
Now, I have the population data, population_by_age_tsv, needed to store in the blob storage manually. And, as the file gets uploaded or updated in the blob storage, I need to build a data flow to capture the change and copy the file to the data lake. To do so, I built a data pipeline using the copy activity used the control flow activities to validate the data file, and enabled the event trigger to detect the file's existence. 

First, I created a linked service for blob storage and data lake storage gen 2 for this copy activity. This will link the dataset and the data pipeline for the copy activity later. 

![LinkedService](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/LinkedService.png)

Then, I added a new dataset for source and sink as below used by the linked services created above steps. 

![DataSetSource](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/DataSetSource.png)
![DataSetSink](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/DataSetSink.png)

After completing both the creation of linked services and datasets, I created a data pipeline for detecting whether there is a file in the blob storage first with a certain logic and performing the copy activity of the source data into the designated data sink location. 

![DataPipeLineGeneral](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/DataPipelineGeneral.png)

This is the overall data pipeline steps...


Validation of the source file existence
![DataPipeLineValidationl](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/DataPipeLineValidation.png)

Get the metadata with exists, size, and column count arguments
![DataPipeLineMetaData](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/DataPipeLineMetaData.png)

Copy the metadata to the data lake and delete the metadata from the source.
![DataPipeLineCopy](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/DataPipeLineIfCondition.png)


### Data Ingestion from HTTP
I have another ECDC dataset coming from Githup. I used HTTP linked service to ingest the dataset from the Githup and copy that to the data lake gen2. 

First, I created another linked service for connecting the dataset to the data pipeline with parameters for the source base URL. 

![LinkedSerivceHTTP](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/LinkedServiceHTTPWithParameters.png)

After creating the HTTP linked service, I created two datasets, one for HTTP source and another for data lake gen 2 sink with parameters. This parameter functionality will reduce the number of datasets and pipelines needed to setup into the ADF. 

For ECDC dataset source, I linked the HTTP linked service and added the base and relative URL parameters. 
![DataSetHTTPSource](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/DataSetHTTPSourceWithParameters.png)
![DataSetHTTPSource](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/DataSetHTTPSinkwithParameters.png)

For ECDC dataset sink, I linked the HTTP linked service and added the sink file name parameter. 
![DataSetHTTPSinkConnetion](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/DataSourceHTTPDataLakeConnection.png)
![DataSetHTTPSinkParameter](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/DataSourceHTTPDataLakeParameters.png)

Up to this point, I successfully created the linked service and two datasets of the ECDC file for the source and sink. Next is to upload a Python code to list each data source of base URL, relative URL, and file name as datasets and create its data pipeline to ingest the HTTP data based on the Python code. 

Below is the Python code list. 
```
[
    {
        "sourceBaseURL":"https://github.com",
        "sourceRelativeURL":"cloudboxacademy/covid19/raw/main/ecdc_data/cases_deaths.csv",
        "sinkFileName":"cases_deaths/cases_deaths.csv"
    },
    {
        "sourceBaseURL":"https://github.com",
        "sourceRelativeURL":"cloudboxacademy/covid19/raw/main/ecdc_data/hospital_admissions.csv",
        "sinkFileName":"hospital_admissions/hospital_admissions.csv"
    },
    {
        "sourceBaseURL":"https://github.com",
        "sourceRelativeURL":"cloudboxacademy/covid19/raw/main/ecdc_data/testing.csv",
        "sinkFileName":"testing/testing.csv"
    },
    {
        "sourceBaseURL":"https://github.com",
        "sourceRelativeURL":"cloudboxacademy/covid19/raw/main/ecdc_data/country_response.csv",
        "sinkFileName":"country_response/country_response.csv"
    }
]
```

![DatasetECDCPythonCode](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/DatasetsECDCPythonCodeList.png)

I created a data ingestion data pipeline for ECDC data containing lookup and ForEach iteration logic to fetch each source in the Python code. Then, performed the copy activity from the blob storage to data lake gen 2. 

![](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/DataPipelineIngestECDCData.png)
![](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/DataPipelineECDCCopySource.png)
![](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/DataPipelineECDCCopySink.png)



### Data flows for Data Transformation for ECDC data

![](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/DataFlowCasesDeaths.png)
![](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/DataFlowHospitalAdm.png)

After ingesting the data files, one for population and another for ECDC, I used the data flows in ADF to perform a data transformation using Select, LookUp, Conditional Split, Join, Pivot, Sort, and Sink. 

### Copy Data to Azure SQL 

![SQLDBLogin!](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/SQLDBLogin.png)
After transforming the datasets, I used the ADF to load them into Azure SQL DB for data analytics. To do so, I went into SQL db to log-in to the Query editor. 

Then, I executed the SQL query below to create each table for cases and deaths, hospital admission, and testing. 

```
CREATE SCHEMA covid_reporting
GO

CREATE TABLE covid_reporting.cases_and_deaths
(
    country                 VARCHAR(100),
    country_code_2_digit    VARCHAR(2),
    country_code_3_digit    VARCHAR(3),
    population              BIGINT,
    cases_count             BIGINT,
    deaths_count            BIGINT,
    reported_date           DATE,
    source                  VARCHAR(500)
)
GO

CREATE TABLE covid_reporting.hospital_admissions_daily
(
    country                 VARCHAR(100),
    country_code_2_digit    VARCHAR(2),
    country_code_3_digit    VARCHAR(3),
    population              BIGINT,
    reported_date           DATE,
    hospital_occupancy_count BIGINT,
    icu_occupancy_count      BIGINT,
    source                  VARCHAR(500)
)
GO

CREATE TABLE covid_reporting.testing
(
    country                 VARCHAR(100),
    country_code_2_digit    VARCHAR(2),
    country_code_3_digit    VARCHAR(3),
    year_week               VARCHAR(8),
    week_start_date         DATE,
    week_end_date           DATE,
    new_cases               BIGINT,
    tests_done              BIGINT,
    population              BIGINT,
    testing_data_source      VARCHAR(500)
)
GO
```

![AzureSQLDB Tables](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/SQLDBTables.png)
With the table creation sql sript, I added the tables with columns into the Azure SQL DB as above. 

The next step was to add the transformed data into the Azure SQL DB using the data copy activity within Azure Data Factory.  

![SQLIZECasesAndDeathSource](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/SQLIZECasesDeathsDataSource.png)
![SQLIZECasesAndDeathSink](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/SQLIZECasesDeathsDataSink.png)
I started with Cases and Deaths data first. 


![SQLIZEHospitalAdmSource](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/SQLIZEHospitalSource.png)
![SQLIZEHospitalAdmSink](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/SQLIZEHospitalSink.png)
Then I performed the copy activity of the Hospital Admissions data later.



After copying the transformed COVID-19 data into the designated SQL tables, I tested the queries as above. 

![CasesAndDeathsData](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/SQLDBCasesAndDeathsTableSelect.png)

![HospitalAdmissionsData](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/SQLDBHospitalAdmissionsTableSelect.png)


### Data Orchestration

Up to this point, I built the data pipelines with triggers enabled containing the data ingestion, data transformation, and data load into the Azure SQL DB individually. In order to perform the data orchestration, I built a data pipeline to combine all the data pipelines with a trigger-tumbling window setup. 

![AllTriggerTumblingWindows](https://github.com/danieljin870711/Azure_Data_Factory_for_Cloud_Engineering_Covid19/blob/main/Screenshots/TriggersSetUpTumblingWindows.png)


