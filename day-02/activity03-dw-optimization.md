# Activity 03: Data Warehouse Optimization

In this activity, you will work with your team to optimize the peformance of a query to make it return results in the shortest time possible. 

# Team Challenge 

WWI wants you to prove your ability to optimize queries using their data. They have provided the following query (where `SUFFIX` is your **student ID**):

``` SQL
SELECT
    FS.CustomerID
    ,MIN(FS.Quantity) as MinQuantity
    ,MAX(FS.Quantity) as MaxQuantity
    ,AVG(FS.Price) as AvgPrice
    ,AVG(FS.TotalAmount) as AvgTotalAmount
    ,AVG(FS.ProfitAmount) as AvgProfitAmount
    ,COUNT(DISTINCT FS.StoreId) as DistinctStores
FROM
    wwi_perf.Sale_Heap_SUFFIX FS
GROUP BY
    FS.CustomerId

```
You are not allowed to modify this table (in fact, **you should not modify this table** as it may affect the other labs). However, they have asked if you can **create a new table** and optimize the query such that it runs much faster than the above one. Working with your team, just how fast can you make this query return results? 

Share your best query time with your learning adviser who will report it back for comparison with the results from other teams.

> NOTE: You can experiment in your individual environments, however when selecting the environment in which to produce your "official" results you should pick the one with the largest SQL Pool size among those available in your Table Group. 

# Activity 03: Data Warehouse Optimization - Solution

1. Check the number of records in the table (where `SUFFIX` is your **student ID**):

```sql
SELECT COUNT(*) FROM [wwi_perf].[Sale_Heap_SUFFIX]
```

2. Check the structure of the existing table:

```sql
CREATE TABLE [wwi_perf].[Sale_Heap_SUFFIX]
( 
	[TransactionId] [uniqueidentifier]  NOT NULL,
	[CustomerId] [int]  NOT NULL,
	[ProductId] [smallint]  NOT NULL,
	[Quantity] [tinyint]  NOT NULL,
	[Price] [decimal](9,2)  NOT NULL,
	[TotalAmount] [decimal](9,2)  NOT NULL,
	[TransactionDateId] [int]  NOT NULL,
	[ProfitAmount] [decimal](9,2)  NOT NULL,
	[Hour] [tinyint]  NOT NULL,
	[Minute] [tinyint]  NOT NULL,
	[StoreId] [smallint]  NOT NULL
)
WITH
(
	DISTRIBUTION = ROUND_ROBIN,
	HEAP
)
```

3. Check the range of `TransactionDateId`:

```sql
SELECT MIN(TransactionDateId), MAX(TransactionDateId) from [wwi_perf].[Sale_Heap_SUFFIX]
```

4. Create the optimized table with `CustomerID`-based hash distribution, Clustered Columnstore Index, and four partitions:

```sql
	
CREATE TABLE [wwi_perf].[Sale_184865_Solution]
WITH
(
	DISTRIBUTION = HASH ( [CustomerId] ),
	CLUSTERED COLUMNSTORE INDEX,
	PARTITION
	(
		[TransactionDateId] RANGE RIGHT FOR VALUES (
            20190101, 20190401, 20190701, 20191001)
	)
)
AS
SELECT
	*
FROM	
	[wwi_perf].[Sale_Heap_184865]
OPTION  (LABEL  = 'CTAS : Sale_184865_Solution')
```