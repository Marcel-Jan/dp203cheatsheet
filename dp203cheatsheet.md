# My Azure DP-203 Cheat Sheet
Things you need to know well for the DP-203 exam.
This is not meant to be exhaustive, just to share the most important highlights.

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



