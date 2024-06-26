# azure-data-pipelines
Azure Data Pipelines Repo

# Step 1: Prepare the Data Infrastructure

## 1. Create the Data Lake and upload data

![Test snapshot](test.png)

## 2. Create an Azure Data Factory Resource

## 3. Create a SQL Database

## 4. Create a Synapse Analytics workspace, or use one you already have created.

## 5. Create summary data external table in Synapse Analytics workspace

```sql
IF NOT EXISTS (SELECT * FROM sys.external_file_formats WHERE name = 'SynapseDelimitedTextFormat') 
	CREATE EXTERNAL FILE FORMAT [SynapseDelimitedTextFormat] 
	WITH ( FORMAT_TYPE = DELIMITEDTEXT ,
	       FORMAT_OPTIONS (
			 FIELD_TERMINATOR = ',',
			 USE_TYPE_DEFAULT = FALSE
			))
GO

IF NOT EXISTS (SELECT * FROM sys.external_data_sources WHERE name = 'adlsnycpayroll-octavian-p_adlsnycpayrolloctavianp_dfs_core_windows_net') 
	CREATE EXTERNAL DATA SOURCE [adlsnycpayroll-octavian-p_adlsnycpayrolloctavianp_dfs_core_windows_net] 
	WITH (
		LOCATION = 'abfss://adlsnycpayroll-octavian-p@adlsnycpayrolloctavianp.dfs.core.windows.net' 
	)
GO
```

```sql
CREATE EXTERNAL TABLE dbo.NYC_Payroll_Summary (
	[FiscalYear] [int] NULL,
    [AgencyName] [varchar](50) NULL,
    [TotalPaid] [float] NULL
	)
	WITH (
	LOCATION = 'dirstaging/nycpayroll_2021.csv',
	DATA_SOURCE = [adlsnycpayroll-octavian-p_adlsnycpayrolloctavianp_dfs_core_windows_net],
	FILE_FORMAT = [SynapseDelimitedTextFormat]
	)
GO
```




## 6. Create master data tables and payroll transaction tables in SQL DB
```sql
CREATE TABLE [dbo].[NYC_Payroll_EMP_MD](
[EmployeeID] [varchar](10) NULL,
[LastName] [varchar](20) NULL,
[FirstName] [varchar](20) NULL
) 
GO
CREATE TABLE [dbo].[NYC_Payroll_TITLE_MD](
[TitleCode] [varchar](10) NULL,
[TitleDescription] [varchar](100) NULL
)
GO
CREATE TABLE [dbo].[NYC_Payroll_AGENCY_MD](
    [AgencyID] [varchar](10) NULL,
    [AgencyName] [varchar](50) NULL
) 
GO
CREATE TABLE [dbo].[NYC_Payroll_Data_2020](
    [FiscalYear] [int] NULL,
    [PayrollNumber] [int] NULL,
    [AgencyID] [varchar](10) NULL,
    [AgencyName] [varchar](50) NULL,
    [EmployeeID] [varchar](10) NULL,
    [LastName] [varchar](20) NULL,
    [FirstName] [varchar](20) NULL,
    [AgencyStartDate] [date] NULL,
    [WorkLocationBorough] [varchar](50) NULL,
    [TitleCode] [varchar](10) NULL,
    [TitleDescription] [varchar](100) NULL,
    [LeaveStatusasofJune30] [varchar](50) NULL,
    [BaseSalary] [float] NULL,
    [PayBasis] [varchar](50) NULL,
    [RegularHours] [float] NULL,
    [RegularGrossPaid] [float] NULL,
    [OTHours] [float] NULL,
    [TotalOTPaid] [float] NULL,
    [TotalOtherPay] [float] NULL
) 
GO

CREATE TABLE [dbo].[NYC_Payroll_Data_2021](
    [FiscalYear] [int] NULL,
    [PayrollNumber] [int] NULL,
    [AgencyCode] [varchar](10) NULL,
    [AgencyName] [varchar](50) NULL,
    [EmployeeID] [varchar](10) NULL,
    [LastName] [varchar](20) NULL,
    [FirstName] [varchar](20) NULL,
    [AgencyStartDate] [date] NULL,
    [WorkLocationBorough] [varchar](50) NULL,
    [TitleCode] [varchar](10) NULL,
    [TitleDescription] [varchar](100) NULL,
    [LeaveStatusasofJune30] [varchar](50) NULL,
    [BaseSalary] [float] NULL,
    [PayBasis] [varchar](50) NULL,
    [RegularHours] [float] NULL,
    [RegularGrossPaid] [float] NULL,
    [OTHours] [float] NULL,
    [TotalOTPaid] [float] NULL,
    [TotalOtherPay] [float] NULL
) 
GO
CREATE TABLE [dbo].[NYC_Payroll_Summary](
    [FiscalYear] [int] NULL,
    [AgencyName] [varchar](50) NULL,
    [TotalPaid] [float] NULL 
)
GO
```


## 7. Screenshot of the below

DataLakeGen2 that shows files are uploaded
![alt text](image.png)

Above 5 tables created in SQL db
![alt text](image-1.png)

External table created in Synapse
![alt text](image-3.png)
![alt text](image-2.png)

# Step 2: Create Linked Services

## 1.Create a Linked Service for Azure Data Lake

In Azure Data Factory, create a linked service to the data lake that contains the data files

From the data stores, select Azure Data Lake Gen 2
Test the connection

## 2. Create a Linked Service to SQL Database that has the current (2021) data
Test the connection


If you get a connection error, remember to add the IP address to the firewall settings in SQL DB in the Azure Portal


## 3.
📝Capture screenshot of Linked Services page after successful creation
![alt text](image-4.png)

📝Save configs of Linked Services after creation



# Step 3: Create Datasets in Azure Data Factory

## 1.Create the datasets for the 2021 Payroll file on Azure Data Lake Gen2

Select DelimitedText
Set the path to the nycpayroll_2021.csv in the Data Lake
Preview the data to make sure it is correctly parsed

## 2. Repeat the same process to create datasets for the rest of the data files in the Data Lake

EmpMaster.csv
TitleMaster.csv
AgencyMaster.csv
Remember to publish all the datasets

## 3. Create the dataset for all the data tables in SQL DB

## 4. Create the datasets for destination (target) table in Synapse Analytics
dataset for NYC_Payroll_Summary

## 5. Snapshots
📝Capture screenshots of datasets in Data Factory
![alt text](image-14.png)

📝Save configs of datasets from Data Factory

# Step 4: Create Data Flows

## 1. Create a new data flow
In Azure Data Factory, create data flow to load 2020 Payroll data from Azure DataLake Gen2 storage to SQL db table created earlier

Create a new data flow
Select the dataset for 2020 payroll file as the source
Click on the + icon at the bottom right of the source, from the options choose sink. A sink will get added in the dataflow
Select the sink dataset as 2020 payroll table created in SQL db
Repeat the same process to add data flow to load data for each file in Azure DataLake to the corresponding SQL DB tables.


## 2. Snapshots

📝Capture screenshots of dataflows in Data Factory
![alt text](image-6.png)
![alt text](image-7.png)

📝Save configs of dataflows from Data Factory



# Step 5: Data Aggregation and Parameterization

In this step, you'll extract the 2021 year data and historical data, merge, aggregate and store it in DataLake staging area which will be used by Synapse Analytics external table. The aggregation will be on Agency Name, Fiscal Year and TotalPaid.

Create new data flow and name it Dataflow Summary
Add source as payroll 2020 data from SQL DB
Add another source as payroll 2021 data from SQL DB
Create a new Union activity and select both payroll datasets as the source
Make sure to do any source to target mappings if required. This can be done by adding a Select activity before Union
After Union, add a Filter activity, go to Expression builder
Create a parameter named- dataflow_param_fiscalyear and give value 2020 or 2021
Include expression to be used for filtering: toInteger(FiscalYear) >= $dataflow_param_fiscalyear
Now, choose Derived Column after filter
Name the column: TotalPaid
Add following expression: RegularGrossPaid + TotalOTPaid+TotalOtherPay
Add an Aggregate activity to the data flow next to the TotalPaid activity
Under Group by, select AgencyName and FiscalYear
Set the expression to sum(TotalPaid)
Add a Sink activity after the Aggregate
Select the sink as summary table created in SQL db
In Settings, tick Truncate table
Add another Sink activity, this will create two sinks after Aggregate
Select the sink as dirstaging in Azure DataLake Gen2 storage
In Settings, tick Clear the folder


📝Capture screenshot of aggregate dataflow in Data Factory
![alt text](image-8.png)

📝Save config of aggregate dataflow from Data Factory



# Step 6: Pipeline Creation
Now, that you have the data flows created it is time to bring the pieces together and orchestrate the flow.

We will create a pipeline to load data from Azure DataLake Gen2 storage in SQL db for individual datasets, perform aggregations and store the summary results back into SQL db destination table and datalake staging storage directory which will be consumed by Synapse Analytics via CETAS.

Create a new pipeline
Include dataflows for Agency, Employee and Title to be parallel
Add dataflows for payroll 2020 and payroll 2021. These should run only after the initial 3 dataflows have completed
After payroll 2020 and payroll 2021 dataflows have completed, dataflow for aggregation should be started.

![alt text](image-9.png)


# Step 7: Trigger and Monitor Pipeline

Select Add trigger option from pipeline view in the toolbar
Choose trigger now to initiate pipeline run
You can go to monitor tab and check the Pipeline Runs
Each dataflow will have an entry in Activity runs list

# Step 8: Verify Pipeline run artifacts

Query data in SQL DB summary table (destination table). This is one of the sinks defined in the pipeline.
Check the dirstaging directory in Datalake if files got created. This is one of the sinks defined in the pipeline
Query data in Synapse external table that points to the dirstaging directory in Datalake.



📝Capture screenshot of pipeline resource from Datafactory
![alt text](image-11.png)

📝Save config of pipeline from Data Factory


📝Capture screenshot of successful pipeline run. All activity runs and dataflow success indicators should be visible
![alt text](image-10.png)

📝Capture screenshot of query from SQL DB summary table
![alt text](image-12.png)

📝Capture screenshot of dirstaging directory listing in Datalake that shows files saved after pipeline runs
![alt text](image-13.png)

📝Capture screenshot of query from Synapse summary external table
![alt text](image-15.png)
