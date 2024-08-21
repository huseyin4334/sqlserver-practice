# Describe Wait Statistics
- Wait statistics are a great way to understand what SQL Server is waiting on. This can help you identify performance bottlenecks and tune your queries accordingly.
- Detecting and troubleshooting SQL Server performance issues require an understanding of how wait statistics work, and how the database engine uses them while processing a request.

Wait statistics are broken down into three types of waits: **resource waits**, **queue waits**, and **external waits**.
- **Resource waits**: 
  - These are waits that occur when a worker thread in SQL Server requests access to a resource that is currently being used by a thread. 
  - Examples of resources wait are locks, latches, and disk I/O waits.
- **Queue waits**: 
  - These are waits that occur when a worker thread is waiting for a resource to become available. 
  - Examples of queue waits are memory, CPU, and scheduler waits.
- **External waits**: 
  - These are waits that occur when a worker thread is waiting for an external process to complete. 
  - Examples of external waits are network, backup, and log waits.
  - An example of an external wait is a network wait related to returning a large result set to a client application.

Some keywords:
- Task: 
  - A task is the basic unit of execution in SQL Server. 
  - Examples of tasks include a query, a login, a logout, and system tasks like ghost cleanup activity, checkpoint activity, log writer, parallel redo activity.
  - https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-os-tasks-transact-sql?view=sql-server-ver16
- Session: A session is a connection to SQL Server that is established by a client application.
- TempDB: 
  - TempDB is a system database in SQL Server that is used to store temporary objects, such as temporary tables, table variables, and temporary stored procedures or created by internal processes.
  - https://learn.microsoft.com/en-us/sql/relational-databases/databases/tempdb-database?view=sql-server-ver16

You can check **sys.dm_os_wait_stats** system view to explore all the waits encountered by threads that executed.
The **sys.dm_exec_session_wait_stats** system view lists active waiting sessions.

As DMVs provide a list of wait types with the highest time accumulated since the last SQL Server startup, collecting and storing wait statistic data periodically could help you understand and correlate performance problems with other database events.

> For example: https://learn.microsoft.com/en-us/training/modules/evaluate-performance-improvements/2-describe-wait-statistics


## Types of Waits
There are several types of waits available in SQL Server, but some of them are common. Here are some of the common wait types:
- **RESOURCE_SEMAPHORE**:
  - This wait type occurs when a query is waiting for memory to become available.
  - May indicate excessive memory grants to some queries.
  - These wait types can be caused by out-of-date statistics, missing indexes, and excessive query concurrency.
- **LCK_M_X**:
  - This wait type occurs when a query is waiting for an exclusive lock on a resource.
  - That can be caused by long-running transactions, missing indexes, and excessive query concurrency.
  - That can be solved by either changing to the **READ COMMITTED SNAPSHOT** isolation level.
  - Making changes in indexing to reduce transaction times, or possibly better transaction management within T-SQL code.
- **PAGEIOLATCH_SH**:
  - This wait type occurs when a query is waiting for a page to be read from disk.
  - This can be caused by slow disk I/O, insufficient memory, or excessive query concurrency.
  - This can be solved by adding more memory to the server, upgrading the disk subsystem, or optimizing queries to reduce the number of reads.
  - if the wait count is low, but the wait time is high, it can indicate storage performance problems.
  - if we read so much data from disk, we can see this wait type in the wait statistics.
- **SOS_SCHEDULER_YIELD**:
  - This wait type occurs when a query is waiting for the scheduler to become available.
  - this wait type can indicate high CPU utilization, which is correlated with either high number of large scans, or missing indexes
  - This can be caused by excessive query concurrency, long-running queries, or insufficient CPU resources.
  - This can be solved by optimizing queries to reduce CPU usage, adding more CPU resources, or reducing query concurrency.
  - Often with high numbers of **CXPACKET** waits.
- **CXPACKET**:
  - This wait type occurs when a query is waiting for another query to complete.
  - if this wait type is high it can indicate improper configuration.
  - This can be caused by parallelism, long-running queries, or insufficient CPU resources.
  - This can be solved by optimizing queries to reduce parallelism, adding more CPU resources, or reducing query concurrency.
- **PAGEIOLATCH_UP**:
  - This wait type occurs when a query is waiting for a page to be written to disk.
  - This wait type on data pages 2:1:1 can indicate TempDB contention on Page Free Space (PFS) data pages.
  - Each data file in TempDB has a PFS page per 64 MB of data that tracks the allocation status of each page in the file.
  - This wait is typically caused by only having one TempDB file, which can cause contention on the PFS pages. This is the default beahvior in sql server 2016.
  - This can be solved by adding more memory to the server, upgrading the disk subsystem, or optimizing queries to reduce the number of writes.


# Tune And Maintain Indexes
A common performance tuning approach is as follows:
- Evaluate existing index usage using **sys.dm_db_index_operational_stats** and **sys.dm_db_index_usage_stats**.
- Consider eliminating unused and duplicate indexes, but this should be done carefully. Some indexes may only be used during monthly/quarterly/annual operations, and may be important for those processes.
- Review and evaluate expensive queries from the Query Store, or Extended Events capture, and work to manually craft indexes to better serve those queries.
- It's important to note any hardware differences between your production and non-production environments, as the amount of memory and the number of CPUs could affect your execution plan.
- After testing carefully, implement the changes to your production system.

> For example: https://learn.microsoft.com/en-us/training/modules/evaluate-performance-improvements/3-tune-and-maintain-indexes


## Resumable Indexes
- Resumable indexes are a new feature in SQL Server 2019 that allows you to pause and resume index operations.
- This feature is useful when you need to perform index maintenance on large tables, but you don't want to lock the table for an extended period of time.

---

We should explain 2 things:
- ONLINE
  - When you create an index with the ONLINE option, SQL Server allows read and write operations to continue on the underlying table while the index is being created.
  - For example, while a clustered index is being rebuilt by one user, that user and others can continue to update and query the underlying data.
  - If we don't use the ONLINE option, the index operation rebuild operation will be blocked until the operation is completed.
  - https://learn.microsoft.com/en-us/sql/relational-databases/indexes/perform-index-operations-online?view=sql-server-ver16
- Resumable
  - Resumable indexes allow you to pause and resume index operations.
  - This feature is useful when you need to perform index maintenance on large tables, but you don't want to lock the table for an extended period of time.
  - For example, if you need to rebuild a large index on a table that is heavily used, you can pause the index operation during peak hours and resume it during off-peak hours.
  - https://learn.microsoft.com/en-us/sql/relational-databases/indexes/guidelines-for-online-index-operations?view=sql-server-ver16#resumable-index-considerations

> Resumable index is only supported with online operations.

---

Index Creation:
```tsql
-- Creates a nonclustered index for the Customer table

CREATE INDEX IX_Customer_PersonID_ModifiedDate 
    ON Sales.Customer (PersonID, StoreID, TerritoryID, AccountNumber, ModifiedDate)
WITH (RESUMABLE=ON, ONLINE=ON)
```

Pausing an Index Operation:
```tsql
-- Pauses the index operation
ALTER TABLE IX_Customer_PersonID_ModifiedDate PAUSE;
```


# Query Hints
- Query hints are a way to override the default behavior of the SQL Server query optimizer.
- They can be used to force the optimizer to use a specific index, join method, or other query plan.
- Query hints are options or strategies that can be applied to enforce the query processor to use a particular operator in the execution plan for `SELECT`, `INSERT`, `UPDATE`, or `DELETE` statements. 
- Query hints override any execution plan the query processor might select for a given query with the `OPTION` clause.

---

Example:
- The `OPTION` clause is used to specify one or more query hints to use with the query.
- The `MAXDOP` hint is used to specify the maximum number of processors that can be used to execute the query.
- The `RECOMPILE` hint is used to the query generates a new, temporary plan each time it's executed.

```tsql
--With maxdop hint
SELECT ProductID, OrderQty, SUM(LineTotal) AS Total  
FROM Sales.SalesOrderDetail
WHERE UnitPrice < $5.00
GROUP BY ProductID, OrderQty  
ORDER BY ProductID, OrderQty  
OPTION (MAXDOP 2)
GO

--With recompile hint
SELECT City
FROM Person.Address
WHERE StateProvinceID=15 OPTION (RECOMPILE)
GO
```

---

Although query hints may provide a localized solution to various performance-related issues, you should avoid using them in production environment for the following reasons.
- You can't benefit from new and improved features in subsequent versions of SQL Server if you bind a query to a specific execution plan.
- Having a permanent query hint on your query can result in structural database changes that would be beneficial to that query not being applicable.


## Query Hints
However, there are several query hints available on SQL Server, which are used for different purposes.
- **FAST <int_value>**: 
  - This hint is used to specify that the query should be optimized for fast retrieval of the first int_value rows.
  - It retrieves the first int_value rows while the query continues to execute.
  - This hint is useful when you want to retrieve the first few rows of a large result set quickly.
  - This hint is equivalent to the `SET ROWCOUNT` statement.
- **OPTIMIZE FOR**: 
  - This hint is used to specify a value for a local variable in a query.
  - Provides instructions to the query optimizer that a particular value for a local variable should be used when a query is compiled and optimized.
  - This hint is useful when you want to optimize the query for a specific value of a local variable.
- **USE_PLAN**: 
  - This hint is used to specify a query plan to use for a query. (the query optimizer will use a query plan specified by the xml_plan attribute)
  - This hint is useful when you want to force the query optimizer to use a specific query plan.
  - This hint is equivalent to the `USE PLAN` statement.
- **RECOMPILE**: 
  - This hint is used to specify that the query should be recompiled each time it's executed.
  - This hint is useful when you want to generate a new, temporary plan each time the query is executed.
- **{LOOP | HASH | MERGE} JOIN**: 
  - This hint is used to specify the join method to use for a query.
  - Specifies all join operations are performed by `LOOP JOIN`, `MERGE JOIN`, or `HASH JOIN` in the whole query. 
  - The optimizer chooses the least expensive join strategy from among the options if you specify more than one join hint.
- **MAXDOP**: 
  - This hint is used to specify the maximum number of processors that can be used to execute the query.
  - This hint is useful when you want to limit the number of processors used to execute the query.


> Also, we can apply multiple query hints to a single query.
> OPTION (HASH GROUP, FAST 10);

https://learn.microsoft.com/en-us/sql/t-sql/queries/hints-transact-sql-query
