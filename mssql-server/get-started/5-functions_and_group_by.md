# Built-in Functions
## Categorize Built-in Functions
- Transact-SQL includes many built-in functions, ranging from functions that perform data type conversion, to functions that aggregate and analyze groups of rows.
- Categories:
  - **Scalar:** Operate on a single row and return a single value.
  - **Logical:** Compare a logical expression and return an appropriate value based on the result.
  - **Ranking:** Operate on a partition (set) of rows and return a ranking value for each row.
  - **Rowset:** Return a virtual table that can be used in a FROM clause in a T-SQL statement.
  - **Aggregate:** Take one or more input values, return a single summarizing value.

> Details: https://learn.microsoft.com/en-us/sql/t-sql/functions/functions?view=sql-server-ver16


## Scalar Functions
- Scalar functions return a single value and usually work on a single row of data.
- The number of input values they take can;
  - Be zero, such as the `GETDATE()` function.
  - Be one, such as the `UPPER()` function.
  - Be more than one, such as the `ROUND()` function.
- The important thing to remember is that scalar functions always return a single value.

- Some considerations when using scalar functions:
  - **Determinism:**  
    - If the function returns the same value for the same input and database state each time it is called, we say it is deterministic. 
    - For example, ROUND(1.1, 0) always returns the value 1.0.
    - Many built-in functions are nondeterministic. For example, GETDATE() returns the current date and time.
  - **Collation:**  
    - The collation of the database can affect the output of some functions. 
    - For example, the UPPER() function returns different results depending on the collation of the database.
    - Some functions use the collation (sort order) of the input value; others use the collation of the database if no input collation is supplied.

### Scalar Function Categories
- At the time of writing, the SQL Server Technical Documentation listed more than 200 scalar functions that span multiple categories, including:
  - Configuration functions
    - @@CONNECTIONS - Returns the number of attempted connections, successful or not, since SQL Server was last started.
    - @@CPU_BUSY - Returns the time in milliseconds that SQL Server has spent performing non-idle tasks.
  - Conversion functions
    - CAST - Converts an expression of one data type to another.
    - CONVERT - Converts an expression of one data type to another.
    - PARSE - Converts a string to a specified data type.
  - Cursor functions
    - CURSOR_STATUS - Returns information about the state of a cursor.
    - FETCH_STATUS - Returns the status of the last fetch operation executed on a cursor.
  - Date and Time functions
    - DATEADD - Adds a specified amount of time to a date.
    - DATEDIFF - Returns the time between two dates.
    - DATEFROMPARTS - Returns a date value for the specified year, month, and day.
  - Mathematical functions
    - ABS - Returns the absolute value of a number.
    - CEILING - Returns
    - DEGREES - Converts radians to degrees.
  - Metadata functions
    - APP_NAME - Returns the name of the application that created the connection.
    - DATABASEPROPERTYEX - Returns the current setting of the specified database property.
    - OBJECT_DEFINITION - Returns the definition of an object.
  - Security functions
    - HAS_PERMS_BY_NAME - Returns 1 if the current user has the specified permission on the specified object.
    - IS_MEMBER - Returns 1 if the current user is a member of the specified database role.
  - String functions
    - CHARINDEX - Returns the starting position of a substring in a string.
    - CONCAT - Concatenates two or more strings.
    - FORMAT - Formats a value with the specified format.
    - LEFT - Returns the left part of a string with the specified number of characters.
  - System functions
    - @@ERROR - Returns the error number for the last T-SQL statement executed.
    - @@IDENTITY - Returns the last identity value generated for an identity column in the same session.
  - System Statistical functions
    - CHECKSUM - Returns the checksum value computed over a row of a table.
    - CHECKSUM_AGG - Returns the checksum of the values in a group.
  - Text and Image functions
    - PATINDEX - Returns the starting position of a pattern in a string.
    - QUOTENAME - Returns a Unicode string with delimiters added to make the input string a valid SQL Server delimited identifier.


### Examples

#### Date and Time Functions

```tsql
SELECT  SalesOrderID,
        OrderDate,
        -- Returns the year of the OrderDate
        YEAR(OrderDate) AS OrderYear,
        -- Returns the name of the month
        DATENAME(mm, OrderDate) AS OrderMonth,
        -- Returns the day of the month
        DAY(OrderDate) AS OrderDay,
        -- Returns the name of the day of the week
        DATENAME(dw, OrderDate) AS OrderWeekDay,
        -- Difference in years between the OrderDate and the current date
        DATEDIFF(yy,OrderDate, GETDATE()) AS YearsSinceOrder
FROM Sales.SalesOrderHeader;
```

| SalesOrderID | OrderDate               | OrderYear | OrderMonth | OrderDay | OrderWeekDay | YearsSinceOrder |
|--------------|-------------------------|-----------|------------|----------|--------------|-----------------|
| 43659        | 2011-05-31 00:00:00.000 | 2011      | May        | 31       | Tuesday      | 10              |


#### Mathematical Functions

```tsql
SELECT TaxAmt,
       -- Returns the rounded value of TaxAmt (10.3234 -> 10.0000) (10.8234 -> 11.0000)
       ROUND(TaxAmt, 0) AS Rounded,
       -- Returns the smallest integer greater than or equal to TaxAmt (10.3234 -> 10.0000)
       FLOOR(TaxAmt) AS Floor,
       -- Returns the largest integer less than or equal to TaxAmt (10.3234 -> 11.0000)
       CEILING(TaxAmt) AS Ceiling,
       -- Returns the value of TaxAmt squared (8 -> 64.0)
       SQUARE(TaxAmt) AS Squared,
       -- Returns the square root of TaxAmt (64 -> 8.0)
       SQRT(TaxAmt) AS Root,
       -- Returns the natural logarithm of TaxAmt (2 -> 0.6931)
       LOG(TaxAmt) AS Log,
       -- Returns the 0 to 1 value from RAND() -> 0.975675675675676
       TaxAmt * RAND() AS Randomized
FROM Sales.SalesOrderHeader;
```

| TaxAmt  | Rounded | Floor | Ceiling | Squared | Root | Log  | Randomized |
|---------|---------|-------|---------|---------|------|------|------------|
| 10.3234 | 10.0000 | 10.0  | 11.0    | 106.4   | 3.21 | 2.33 | 10.075     |


#### String Functions

```tsql
SELECT  CompanyName,
        UPPER(CompanyName) AS UpperCase,
        LOWER(CompanyName) AS LowerCase,
        LEN(CompanyName) AS Length,
        REVERSE(CompanyName) AS Reversed,
        CHARINDEX(' ', CompanyName) AS FirstSpace,
        LEFT(CompanyName, CHARINDEX(' ', CompanyName)) AS FirstWord,
        -- Starting from the first space, return the rest of the string
        SUBSTRING(CompanyName, CHARINDEX(' ', CompanyName) + 1, LEN(CompanyName)) AS RestOfName
FROM Sales.Customer;
```

| CompanyName     | UpperCase       | LowerCase       | Length | Reversed        | FirstSpace | FirstWord | RestOfName |
|-----------------|-----------------|-----------------|--------|-----------------|------------|-----------|------------|
| Adventure Works | ADVENTURE WORKS | adventure works | 14     | skroW erutnevdA | 10         | Adventure | Works      |


## Logical Functions
- Logical functions evaluate an input expression, and return an appropriate value based on the result.

### IIF
- The `IIF` function evaluates an expression and returns one of two values, depending on whether the expression evaluates to true or false.
- The syntax is `IIF (boolean_expression, true_value, false_value)`.

```tsql
SELECT  SalesOrderID,
        OrderDate,
        -- If the OrderDate is before 2012-01-01, return 'Old Order', otherwise return 'New Order'
        IIF(OrderDate < '2012-01-01', 'Old Order', 'New Order') AS OrderType
FROM Sales.SalesOrderHeader;


SELECT AddressType,
       -- If the AddressType is 'Main Office', return 'Billing', otherwise return 'Mailing'
       IIF(AddressType = 'Main Office', 'Billing', 'Mailing') AS UseAddressFor
FROM Sales.CustomerAddress;
```

### CHOOSE
- The `CHOOSE` function returns the value at the specified index from a list of values.
- Choose is a 1-based function, meaning the index starts at 1.
- If the index is bigger than the number of values, the function returns NULL.

```tsql
SELECT  SalesOrderID,
        OrderDate,
        -- Returns the name of the month based on the month number
        CHOOSE(MONTH(OrderDate), 'January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December') AS OrderMonth
FROM Sales.SalesOrderHeader;
```

| SalesOrderID | OrderDate               | OrderMonth |
|--------------|-------------------------|------------|
| 43659        | 2011-05-31 00:00:00.000 | May        |


## Ranking Functions
- Ranking functions allow you to perform calculations against a user-defined set of rows. 
- These functions accept a set of rows as input and return a set of rows as output.
- These functions include ranking, offset, aggregate, and distribution functions.

- Ranking functions include:
  - **ROW_NUMBER:** Returns a unique number for each row in the result set.
  - **RANK:** Returns the rank of each row in the result set.
  - **DENSE_RANK:** Returns the rank of each row in the result set without gaps.
    - Gap is the difference between the rank of the current row and the rank of the previous row.
    - Without gaps, the rank of the current row is the same as the rank of the previous row.
  - **PERCENT_RANK:** Returns the relative rank of each row in the result set.
  - **CUME_DIST:** Returns the cumulative distribution of each row in the result set.
- Some Keywords:
  - **PARTITION BY:** Divides the result set into partitions to which the function is applied.
  - **ORDER BY:** Specifies the order of the rows in each partition.
  - **OVER:** Specifies the partitioning and ordering of the row set.
    - More: https://learn.microsoft.com/en-us/sql/t-sql/queries/select-over-clause-transact-sql?view=sql-server-ver16

```tsql
SELECT  SalesOrderID,
        OrderDate,
        -- Returns the row number for each row in the result set
        ROW_NUMBER() OVER (ORDER BY OrderDate) AS RowNumber,
        -- Returns the rank of each row in the result set
        RANK() OVER (ORDER BY OrderDate) AS Rank,
        -- Returns the dense rank of each row in the result set
        DENSE_RANK() OVER (ORDER BY OrderDate) AS DenseRank,
        -- Returns the percent rank of each row in the result set
        PERCENT_RANK() OVER (ORDER BY OrderDate) AS PercentRank,
        -- Returns the cumulative distribution of each row in the result set
        CUME_DIST() OVER (ORDER BY OrderDate) AS CumulativeDistribution
FROM Sales.SalesOrderHeader;
```

> Details: https://learn.microsoft.com/en-us/sql/t-sql/functions/ranking-functions-transact-sql?view=sql-server-ver16


### Examples

This example uses the RANK function to calculate a ranking based on the ListPrice, with the highest price ranked at 1

```tsql
SELECT TOP 100 ProductID, Name, ListPrice,
               RANK() OVER(ORDER BY ListPrice DESC) AS RankByPrice
FROM Production.Product AS p
ORDER BY RankByPrice;
```

| ProductID | Name             | ListPrice | RankByPrice |
|-----------|------------------|-----------|-------------|
| 709       | Road-650 Red, 52 | 3578.27   | 1           |
| 708       | Road-650 Red, 48 | 3578.27   | 1           |
| 707       | Road-650 Red, 44 | 3399.99   | 4           |


---

- **PARTITION BY** divides the result set into partitions.
- **ORDER BY** that defines the logical order of the rows within each partition of the result set.
- **ROWS/RANGE** that limits the rows within the partition by specifying start and end points within the partition. It requires ORDER BY argument and the default value is from the start of partition to the current element if the ORDER BY argument is specified.

```tsql
SELECT c.Name AS Category, p.Name AS Product, ListPrice,
  RANK() OVER(PARTITION BY c.Name ORDER BY ListPrice DESC) AS RankByPrice
FROM Production.Product AS p
JOIN Production.ProductCategory AS c
ON p.ProductCategoryID = c.ProductcategoryID
ORDER BY Category, RankByPrice;
```

| Category           | Product                   | ListPrice | RankByPrice |
|--------------------|---------------------------|-----------|-------------|
| Accessories        | HL Road Frame - Black, 58 | 1431.50   | 1           |
| Accessories        | HL Road Frame - Red, 58   | 1431.50   | 1           |
| Accessories        | HL Road Frame - Red, 62   | 14334.50  | 2           |
| Accessories-Others | All-Purpose Bike Stand    | 89.99     | 1           |
| Accessories-Others | All-Purpose Bike Stand    | 90.99     | 2           |


## Rowset Functions
- Rowset functions return a virtual table that can be used in the FROM clause as a data source. 
- These functions take parameters specific to the rowset function itself.

- Rowset functions include:
  - **OPENROWSET:** Provides ad hoc connection information for accessing remote data from an OLE DB data source.
  - **OPENQUERY:** Executes the specified pass-through query on the specified linked server.
  - **OPENXML:** Provides a rowset view over an XML document.
  - **OPENDATASOURCE:** Provides ad hoc connection information for accessing remote data from an OLE DB data source.
  - **OPENJSON:** Returns a rowset view over a JSON document.
- The OPENDATASOURCE, OPENQUERY, and OPENROWSET functions enable you to pass a query to a remote database server.


```tsql
SELECT a.*
FROM OPENROWSET('SQLNCLI', 'Server=SalesDB;Trusted_Connection=yes;',
    'SELECT Name, ListPrice
    FROM AdventureWorks.Production.Product') AS a;
```


## Aggregate Functions
- Aggregate functions take one or more input values and return a single summarizing value.
- T-SQL provides aggregate functions such as SUM, MAX, and AVG to perform calculations that take multiple values and return a single result.
- These functions can be used in SELECT, HAVING, and ORDER BY clauses. But not in WHERE clause.
- Aggregate functions ignore NULL values. Except for the COUNT function, which counts all rows, regardless of NULL values.
- Unless you're using GROUP BY, you shouldn't combine aggregate functions with columns not included in functions in the same SELECT list.

> Note: To extend beyond the built-in functions, SQL Server provides a mechanism for user-defined aggregate functions via the .NET Common Language Runtime (CLR). That topic is beyond the scope of this module.

- Functions:
  - **AVG:** Returns the average of the values in a group.
  - **COUNT:** Returns the number of items in a group.
  - **MAX:** Returns the maximum value in a group.
  - **MIN:** Returns the minimum value in a group.
  - **SUM:** Returns the sum of the values in a group.
  - **COUNT_BIG:** Returns the number of items in a group, with a bigint return type.

---
```tsql
-- Returns the average, minimum, and maximum list prices for all products
SELECT AVG(ListPrice) AS AveragePrice,
       MIN(ListPrice) AS MinimumPrice,
       MAX(ListPrice) AS MaximumPrice
FROM Production.Product;

-- Returns the average, minimum, and maximum list prices for products in category 15 (We grouped by ProductCategoryID)
SELECT AVG(ListPrice) AS AveragePrice,
       MIN(ListPrice) AS MinimumPrice,
       MAX(ListPrice) AS MaximumPrice
FROM Production.Product
WHERE ProductCategoryID = 15;
```

---

```tsql
-- This query will take error because of we have to either use aggregate functions in all select list parameter or use GROUP BY clause.
/*
    Msg 8120, Level 16, State 1, Line 1
    Column 'Production.ProductCategoryID' is invalid in the select list because it isn't contained in either an aggregate function or the GROUP BY clause.
 */
SELECT ProductCategoryID, AVG(ListPrice) AS AveragePrice,
       MIN(ListPrice) AS MinimumPrice,
       MAX(ListPrice) AS MaximumPrice
FROM Production.Product;

-- This query will work because we grouped by ProductCategoryID
SELECT ProductCategoryID, AVG(ListPrice) AS AveragePrice,
MIN(ListPrice) AS MinimumPrice,
MAX(ListPrice) AS MaximumPrice
FROM Production.Product;
GROUP BY ProductCategoryID;
```

---

- The MIN and MAX functions can also be used with date data, to return the earliest and latest chronological values. 
- However, AVG and SUM can only be used for numeric data, which includes integers, money, float and decimal datatypes.

```tsql
SELECT MIN(CompanyName) AS MinCustomer, 
       MAX(CompanyName) AS MaxCustomer
FROM SalesLT.Customer;

-- | MinCustomer | MaxCustomer |
-- |-------------|-------------|
-- | A Bike Store | Wingtip Toys |


SELECT MIN(YEAR(OrderDate)) AS Earliest,
       MAX(YEAR(OrderDate)) AS Latest
FROM Sales.SalesOrderHeader;

-- | Earliest | Latest |
-- |----------|--------|
-- | 2011     | 2014   |
```

> NOTE: The presence of NULLs in a column may lead to inaccurate computations for AVG, which will sum only populated rows and divide that sum by the number of non-NULL rows. 
> There may be a difference in results between AVG(<column>) and (SUM(<column>)/COUNT(*)).

```tsql
SELECT SUM(c2) AS sum_nonnulls, 
    COUNT(*) AS count_all_rows, 
    COUNT(c2) AS count_nonnulls, 
    AVG(c2) AS average, 
    (SUM(c2)/COUNT(*)) AS arith_average
FROM t1;
```

| sum_nonnulls | count_all_rows | count_nonnulls | average | arith_average |
|--------------|----------------|----------------|---------|---------------|
| 150          | 6              | 5              | 30      | 25            |



# Group By
- The GROUP BY clause groups rows that have the same values into summary rows, like "find the number of customers in each country".
- when your SELECT statement is processed, after the FROM clause and WHERE clause have been evaluated, a virtual table is created. 
- The contents of the virtual table are now available for further processing. 
- You can use the GROUP BY clause to subdivide the contents of this virtual table into groups of rows.
- `GROUP BY <value1> [, <value2>, …]`

```tsql
SELECT CustomerID
FROM Sales.SalesOrderHeader
GROUP BY CustomerID;

-- This query equivalent to the previous query
SELECT DISTINCT CustomerID
FROM Sales.SalesOrderHeader;
```

- The difference between the group by and distinct is that the group by can be used with aggregate functions.

```tsql
SELECT CustomerID, COUNT(*) AS OrderCount
FROM Sales.SalesOrderHeader
GROUP BY CustomerID;
```
- Count by CustomerID

| CustomerID | OrderCount |
|------------|------------|
| 29825      | 3          |
| 29826      | 5          |

- Performed the query following this order;
  - FROM 
  - WHERE
  - GROUP BY
  - HAVING
  - SELECT
  - ORDER BY

---

- Because of that, we have to consider the column names;
  - Order by can see the column names in the select list.
  - But Group by can not see the aliases in the select list.

```tsql
SELECT CustomerID AS Customer,
       COUNT(*) AS OrderCount
FROM Sales.SalesOrderHeader
GROUP BY CustomerID
ORDER BY Customer;
```

## Troubleshooting
- This means, You can not <column_name> in the select list because it is not contained in either an aggregate function or the GROUP BY clause.

```tsql
/*
    Msg 8120, Level 16, State 1, Line 2
    Column <column_name> is invalid in the select list because it is not contained in either an aggregate function or the GROUP BY clause.
 */
 
SELECT CustomerID, PurchaseOrderNumber, COUNT(*) AS OrderCount
FROM Sales.SalesOrderHeader
GROUP BY CustomerID;
```

## Having
- The HAVING clause is used to filter groups of rows.
- A HAVING clause enables you to create a search condition, conceptually similar to the predicate of a WHERE clause, which then tests each group returned by the GROUP BY clause.

```tsql
-- Find the number of customers who have placed more than 10 orders
SELECT CustomerID,
      COUNT(*) AS OrderCount
FROM Sales.SalesOrderHeader
GROUP BY CustomerID
HAVING COUNT(*) > 10;
```








