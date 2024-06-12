# azure-data-pipelines
Azure Data Pipelines Repo

 1. Create the Data Lake and upload data

![alt text](image.png)

```sql
CREATE EXTERNAL TABLE [dbo].[NYC_Payroll_Summary](
[FiscalYear] [int] NULL,
[AgencyName] [varchar](50) NULL,
[TotalPaid] [float] NULL
)
WITH (
LOCATION = '/',
DATA_SOURCE = [mydlsfs20230413_mydls20230413_dfs_core_windows_net],
FILE_FORMAT = [SynapseDelimitedTextFormat]
)
GO
```