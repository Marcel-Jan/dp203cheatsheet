# My Azure DP-203 Cheat Sheet
Things you need to know well for the DP-203 exam.  
This is not meant to be exhaustive, just to share the most important highlights.  

Also check out this Cheat Sheet for dedicated SQL pool in Azure Synapse Analytics:  
[https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/cheat-sheet](https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/cheat-sheet)


# Windowing functions
Just a great topic that I imagine writers of exam questions love. Here is most of what you need to know.

| Type | Contiguous? | Overlapping? | Window size | Window starts when.. | Details |
| ---- | ----------- | ------------ | ----------- | -------------------- | ------- |
| Tumbling | Yes | No | Fixed | At fixed time | }
| Hopping | No | Yes | Fixed | Fixed | |
| Sliding | No | Yes | Fixed? | When event enters/exits | |
| Session | No | No | Variable | When first event occurs | |
| Snapshot | No | No window | N/A | N/A | Groups events that have the same timestamp |

More info:
[https://learn.microsoft.com/en-us/azure/stream-analytics/stream-analytics-window-functions](https://learn.microsoft.com/en-us/azure/stream-analytics/stream-analytics-window-functions)


# External tables
Important topics to know
* You need create an external data source, external file format and then you can create an external table.
* When you set LOCATION for an external table, what files are included and what won't be read? What do wildcards do? What will it do with hidden files.

## LOCATION
What files will be read? That depends on the used technology.

| Location definition | Type of env | Will read subdirs? | Will read hidden files? | Will read hidden dirs? |
| ------------------- | ------- | ---- | ---- | -- |
| LOCATION='/webdata' | Polybase | Yes | No | No |
| LOCATION='/webdata' | Hadoop | Yes | No | No |
| LOCATION='/webdata' | Native external table | No | No | No |
| LOCATION='/webdata/**' | Native external table | Yes | No | No |

Hidden files and folders: "Just like Hadoop, PolyBase doesn't return hidden folders. It also doesn't return files for which the file name begins with an underline (_) or a period (.)."  
(PolyBase is a technology that accesses external data stored in Azure Blob storage or Azure Data Lake Store via the T-SQL language.)  

https://learn.microsoft.com/en-us/sql/t-sql/statements/create-external-table-transact-sql

Suppose we have this structure:  
| | | |
| ---- | --- | --- |
| / webdata | / mydata1.txt | |
|           | / month | / mydata2.txt |
|         | / .hiddenfolder | / mydata3.txt |
|         | / _hidden.txt |  |

On an Hadoop external table it will read mydata1.txt and mydata2.txt , but not mydata3.txt or _hidden.txt.  
On a native external table it will only read mydata1.txt, unless wildcards are used.  
Note: Serverless SQL pools only have access to native external tables. So when there is no wildcard in LOCATION, you'll know what to expect.  

More info:
[https://learn.microsoft.com/en-us/azure/synapse-analytics/sql/develop-tables-external-tables](https://learn.microsoft.com/en-us/azure/synapse-analytics/sql/develop-tables-external-tables)


In Serverless SQL pools you can do adhoc queries on files. Know that the FORMAT option in OPENROWSET has no 'JSON' choice. In that case, choose 'CSV'.  


## File formats
We're talking about file formats in external tables/ data lakes  
Parquet is often the preferred file format in data lakes.  

| File Format | Binary | Orientation | Schema defined |
| -- | -- | -- | -- |
| Parquet | Yes | Columnar | Yes |
| ORC | Yes | Columnar | |
| AVRO | Yes | Row | Yes |
| CSV | No | Row | No |
| JSON | No | Row | No |
| XML | No | Row | No |

Columnar: when you read a file with 40 rows and query only 4 rows, a columnar file format will only need to load the 4 columns.  
Row: file is grown row by row.
Also important to know: AVRO supports timestamps.  

While Parquet is usually the best answer for performance, in Azure Data Lake Gen2 there's a feature called query acceleration with which you can accelerate the read performance of CSV and JSON files.  


## Access to files in Data Lake Storage
This is about accessing folders and files with Access Control Lists (ACL) and having Role-Based Access Controls (RBAC) on them. You'll also frequently see the term "least privileges" mentioned. This means you only get privileges for the stuff you need to do, but no more.  
When you need to access files in a folder you need to know what permissions you need in the underlying folders. It is very similar how this stuff works in Linux (POSIX).  
If you need to traverse folders to get to files in them, you need Execute permissions on that folder.  

Example:  
Suppose there's a container called mycontainer. And it has a folder called myfolder. And in the folder are the files you need to read, like myfile.txt.   
In that case you need to have:
* Execute permissions on mycontainer.
* Execute permissions on myfolder.
* To read myfile.txt you need Read permissions.

If you need to list the contents of myfolder you need to add this permission:
* Read permission on myfolder.

If you need to write files in myfolder, you also need this permission:
* Write permissions on myfolder.

Other scenarios are described in this document:  
[https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-access-control#levels-of-permission](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-access-control#levels-of-permission)


# (Databricks) Auto loader
Used to incrementally load cloud data into the delta lake. Good for streaming data.  
You can do so without specifying a specific schema.  
[https://learn.microsoft.com/en-us/azure/databricks/ingestion/cloud-object-storage/auto-loader/](https://learn.microsoft.com/en-us/azure/databricks/ingestion/cloud-object-storage/auto-loader/)


# Data modelling
There are staging, dimension and fact tables.  

Fact tables store observations or events, and can be sales orders, stock balances, exchange rates, temperatures, and more. A fact table contains dimension key columns that relate to dimension tables, and numeric measure columns.  
Dimension tables describe business entities—the things you model. Entities can include products, people, places, and concepts including time itself.  

A surrogate key is a unique identifier that you add to a table to support star schema modeling. By definition, it's not defined or stored in the source data. Commonly, surrogate keys are added to relational data warehouse dimension tables to provide a unique identifier for each dimension table row.

Basically, read this document very well:  
[https://learn.microsoft.com/en-us/power-bi/guidance/star-schema](https://learn.microsoft.com/en-us/power-bi/guidance/star-schema)

Dimension tables:
* Things you model (products, people, places, concepts)
* Usually 2NF, possibly 3NF, but not 1NF
* Primary key: IDENTITY column.

Fact tables:
* Observations or events (sales orders, stock balances, exchange rates, temperatures, etc.)



# Distribution and partitioning
You do distribution because: you want to distribute data over (60) nodes (in dedicated SQL pools), to divide the workload over those nodes.  
Sometimes, when you have smaller tables (dimensions!) you want to have that same table on all those nodes.  

You do partitioning so you can get subsets of a big table without having to read it all. Imagine a big sales table. You mostly query it on date. So when you partion your sales table on the data, and you do a WHERE salesdate = '20241223', only that partition is read and not the whole table.  
Partitioning only works when you do your selects on that partitioned column.  

## Partitioning strategy
For optimal compression and performance of clustered columnstore tables, a minimum of 1 million rows per distribution and partition is needed.  
Example:  
If you got a table of 4.8 billion rows, devide that by 60 nodes (distribution) = 80 million rows per node.  
If you choose your partition strategy so that you have 80 partitions, that would result in a million rows per distribution. Which would be optimal.  

In an MPP system, the data in a table is distributed for processing across a pool of nodes. Synapse Analytics supports the following kinds of distribution:  
- Hash: A deterministic hash value is calculated for the specified column and used to assign the row to a compute node.  
- Round-robin: Rows are distributed evenly across all compute nodes.  
- Replicated: A copy of the table is stored on each compute node.  
[https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-distribute](https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-distribute)


| Table type | Recommended distribution option |
| -- | -- |
| Dimension | Use replicated distribution for smaller tables to avoid data shuffling when joining to distributed fact tables. If tables are too large to store on each compute node, use hash distribution. |
| Fact | Use hash distribution with clustered columnstore index to distribute fact tables across compute nodes. |
| Staging | Use round-robin distribution for staging tables to evenly distribute data across compute nodes. |

![distribution-distribution](https://github.com/user-attachments/assets/1efc2173-b47d-4fb2-8eff-e0fca4b1dc74)


Recommendations from:
[https://learn.microsoft.com/en-us/azure/synapse-analytics/sql/develop-tables-overview](https://learn.microsoft.com/en-us/azure/synapse-analytics/sql/develop-tables-overview)

But what is a "smaller table"? About 2G (after compression) it seems.  

It is also important to know what column you decide to distribute on. You need to choose a distribution that distributes the data evenly.

Good choice for a distribution column (even distribution):
* Lots of unique values
* Does not have a lot of NULLs
* Not a date column

Good choice for a distribution column (minimized data movement):
* Is used in JOINs, GROUP BY, OVER, HAVING.
* NOT used in WHERE clauses.
* Not a date column.

https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-distribute#choose-a-distribution-column

## Skew
When you don't pick a good distribution column you can get data skew. You need to know how to detect it.
``` DBCC PDW_SHOWSPACEUSED('dbo.FactInternetSales'); ```
Make sure you run it in the correct pool.  

## Indexing
And then there's indexing.
Staging tables benefit from heap tables. A heap is a table without a clustered index.
For the rest clustered column index is usually a good choice.

Will it improve load times?  
compressing: Yes  
columnstore: No  

### Rebuilding indexes
Apparently it is necessary sometimes to rebuild indexes.  
"If you frequently perform INSERT, UPDATE, or DELETE operations on a heap table, it's advisable to include table rebuilding in your maintenance schedule by using ALTER TABLE command. For example, ALTER TABLE [SchemaName].[TableName] REBUILD. This practice contributes to reduced fragmentation, resulting in improved performance during read operations."  
[https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-index](https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-index)


# Storage temperatures
Use archive storage only when you're not going to access it.

Check the table here for when to choose hot, cold or archive storage:  
[https://learn.microsoft.com/en-us/azure/storage/blobs/access-tiers-overview#summary-of-access-tier-options](https://learn.microsoft.com/en-us/azure/storage/blobs/access-tiers-overview#summary-of-access-tier-options)


# Data masking
You need to know when data masking is a viable solution.
Data masking replaces sensitive data in columns by masked data. So users still can read the table / file, but they get to see masked data.

Masking is done with a masking function. Creating a table with masking looks like this:

```
CREATE TABLE Data.Membership (
    MemberID INT IDENTITY(1, 1) NOT NULL,
    FirstName VARCHAR(100) MASKED WITH (FUNCTION = 'partial(1, "xxxxx", 1)') NULL,
    LastName VARCHAR(100) NOT NULL,
    Phone VARCHAR(12) MASKED WITH (FUNCTION = 'default()') NULL,
    Email VARCHAR(100) MASKED WITH (FUNCTION = 'email()') NOT NULL,
    DiscountCode SMALLINT MASKED WITH (FUNCTION = 'random(1, 100)') NULL,
    BirthDay DATETIME MASKED WITH (FUNCTION = 'default()') NULL
);
```

You need to do what the default() function does with data:

| Function | Type of data | Original value | Masked value |
| -- | -- | -- | -- |
| default() | ints, floats, real, money | 1433.29 | 0 |
| default() | nvarchar, nchar, ntext | Marcel-Jan Krijgsman | XXXX |
| default() | date, datetime2, time | 2025-01-23 | 1900-01-01 |
| default() | timestamp, table, and others | | <empty> |
| email() | mail address | dp203@alltheanswers.com | dXX@XXXX.com |

There also is a creditcard function and you can mask parts of data with random(1, 100) and partial(1, "xxxxx", 1).  

Users with administrative rights like server admin, Microsoft Entra admin, and db_owner role can view the original data without any mask. (Note: It also applies to sysadmin role in SQL Server)  
[https://docs.microsoft.com/en-us/azure/azure-sql/database/dynamic-data-masking-overview](https://docs.microsoft.com/en-us/azure/azure-sql/database/dynamic-data-masking-overview)


# Temporal tables
Not sure this is part of the requirements for DP-203, but I just saw this passing by as possible solution.

A temporal table is a system versioned table. If you have rows that you regularly update and you want to keep track of earlier versions, this could be a way to do it.

```
CREATE TABLE cvgen.ResumeIntroduction
(
     IntroductionId INT PRIMARY KEY IDENTITY(1,1),
	 IntroductionText NVARCHAR(MAX),
     SysStartTime datetime2 GENERATED ALWAYS AS ROW START NOT NULL,
     SysEndTime datetime2 GENERATED ALWAYS AS ROW END NOT NULL,
     PERIOD FOR SYSTEM_TIME (SysStartTime,SysEndTime)
)
WITH (SYSTEM_VERSIONING = ON);
```

Temporal tables are not available in serverless SQL pools.


# Storage and zones, regions
Read this doc:  
[https://learn.microsoft.com/en-us/azure/storage/common/storage-redundancy](https://learn.microsoft.com/en-us/azure/storage/common/storage-redundancy)
