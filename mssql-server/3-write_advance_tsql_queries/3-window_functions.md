# Window Functions
- Window functions allow you to perform calculations such as ranking, aggregations, and offset comparisons between rows.
- Window functions are used to perform calculations across a set of table rows that are related to the current row. (set of rows -> window)
- The difference between window functions and built-in functions is that window functions operate on a set of rows and return a single value for each row from the underlying query.
- The **OVER** clause is used to define the window you want to work on.

Window functions solve common problems such as generating row numbers in a result set or calculating running totals. 
Windows also provide an efficient way to compare values in one row with values in another without needing to join a table to itself.


## Over Clause
- The OVER clause defines the rows that the window function is applied to. This may be all the rows, or a subset of the rows. 
- It can also define the order of the rows for a window function.

The OVER clause can take the following arguments:
- **PARTITION BY**: Divides up the result set into partitions before applying the window function. The window function is applied to each partition separately.
- **ORDER BY**: Specifies the order of the rows in each partition.
- **ROWS/RANGE**: Specifies the window frame within a partition.
  - The ROW or RANGE arguments set a start and end boundary around the rows being operated on. ROW or RANGE requires an ORDER BY subclause within the OVER clause.
  - ROWS: Specifies a physical offset from the current row.
  - RANGE: Specifies a logical offset from the current row.
  - **BETWEEN AND**: Specifies the start and end of the window frame.

If you don't specify an argument to the OVER clause, the window functions will be applied on the entire result set.


## Functions
- **Aggregate Functions**: Such as SUM, AVG, and COUNT which operate on a window and return a scalar value.
- **Ranking Functions**: Such as ROW_NUMBER, RANK, DENSE_RANK, and NTILE which assign a rank to each row in a partition.
- **Analytic Functions**: Such as CUME_DIST, PERCENTILE_CONT, or PERCENTILE_DISC. Analytic functions calculate the distribution of values in the partition.
- **Offset Functions**: Such as LAG, LEAD, FIRST_VALUE, and LAST_VALUE. Offset functions return values from other rows relative to the position of the current row.

---

- Aggregate Functions

```sql
SELECT Name, ProductNumber, Color, SUM(Weight) 
OVER(PARTITION BY Color) AS WeightByColor
FROM SalesLT.Product
ORDER BY ProductNumber;
```

| Name                      | ProductNumber | Color | WeightByColor |
|---------------------------|---------------|-------|---------------|
| HL Road Frame - Black, 58 | FR-R92B-58    | Black | 110.00        |
| HL Road Frame             | FR-R92R-58    | Red   | 105.00        |
| HL Road Frame - Red, 58   | FR-R92R-58    | Red   | 105.00        |
| Sport-100 Helmet, Red     | HL-U509-R     | Red   | 105.00        |


---

- Ranking Functions
- ROW_NUMBER(): Assigns a unique sequential integer to each row within a partition of a result set.
- RANK(): Assigns a unique integer to each distinct row within a partition of a result set.
- DENSE_RANK(): Assigns a unique integer to each distinct row within a partition of a result set, but does not leave gaps between the ranks.
- NTILE(): Divides the result set into a specified number of groups, or buckets, and assigns a bucket number to each row.

```sql
SELECT productid, name, listprice 
    ,ROW_NUMBER() OVER (ORDER BY productid) AS "Row Number"  
    ,RANK() OVER (ORDER BY listprice) AS PriceRank  
    ,DENSE_RANK() OVER (ORDER BY listprice) AS "Dense Rank"  -- Dense rank uses for duplicate values
    ,NTILE(4) OVER (ORDER BY listprice) AS Quartile  -- Divide the result set into 4 equal parts
FROM SalesLT.Product;
```

| productid | name                      | listprice | Row Number | PriceRank | Dense Rank | Quartile |
|-----------|---------------------------|-----------|------------|-----------|------------|----------|
| 680       | HL Road Frame - Black, 58 | 1431.50   | 1          | 1         | 1          | 1        |
| 681       | HL Road Frame             | 1431.50   | 2          | 1         | 1          | 1        |
| 707       | HL Road Frame - Red, 58   | 1.50      | 10         | 2         | 2          | 1        |
| 685       | Sport-100 Helmet, Red     | 34.99     | 6          | 5         | 3          | 1        |


---

- Analytic Functions
- CUME_DIST(): Returns the cumulative distribution of a value within a group of values.
- PERCENTILE_CONT(): Returns a value that corresponds to a specified percentile within a group of values.
- PERCENT_RANK(): Returns the relative rank of a value within a group of values.
- FIRST_VALUE(): Returns the first value in an ordered set of values.

---

- Offset Functions
- Lag and Lead functions are used to access data from a previous or subsequent row in the same result set without the use of a self-join. This requires an ORDER BY clause.
- LAG(): Returns the value of an expression from the previous row.
- LEAD(): Returns the value of an expression from the next row.

- FIRST_VALUE(): Returns the first value in an ordered set of values.
- LAST_VALUE(): Returns the last value in an ordered set of values.

```tsql
-- Scalar expression is the column name
-- offset is the number of rows to go back
-- default is the value to return if the offset goes beyond the first row.
LAG (scalar_expression [,offset] [,default])
    OVER ( [ partition_by_clause ] order_by_clause )
```

```tsql
-- Budget for the current year and the budget for the previous year
SELECT YEAR, BUDGET, LAG(BUDGET, 1, 0) OVER (ORDER BY YEAR) AS PreviousYearBudget
FROM Sales.SalesPersonQuotaHistory
WHERE BusinessEntityID = 275;
```

- Budget is 300000 for 2005 and previous year budget is 0. Because we set the default value to 0.
- Budget is 250000 for 2006 and previous year budget is 300000.

| YEAR | BUDGET | PreviousYearBudget |
|------|--------|--------------------|
| 2005 | 300000 | 0                  |
| 2006 | 250000 | 300000             |
| 2007 | 250000 | 250000             |


```tsql
-- Budget for the current year and the budget for the previous year
SELECT YEAR, BUDGET, LEAD(BUDGET, 1, 0) OVER (ORDER BY YEAR) AS PreviousYearBudget
FROM Sales.SalesPersonQuotaHistory
WHERE BusinessEntityID = 275;
```

| YEAR | BUDGET | PreviousYearBudget |
|------|--------|--------------------|
| 2005 | 300000 | 250000             |
| 2006 | 250000 | 500000             |
| 2007 | 500000 | 250000             |
















