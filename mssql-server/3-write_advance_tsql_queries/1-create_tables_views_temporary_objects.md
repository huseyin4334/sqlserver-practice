# Create Tables

## Point Database
- We can point to the database using the `USE` statement.
- When we use this statement, all the subsequent queries will be executed in the context of the database we have pointed to.
- This statement covers the entire session.

```tsql
USE AdventureWorks2019;
```

## Create Table

- We can create tables using the `CREATE TABLE` statement.

```tsql
CREATE TABLE Products  
(ProductID int PRIMARY KEY NOT NULL,  
ProductName varchar(50) NOT NULL,  
ProductDescription varchar(max) NOT NULL);
```

> You must have the CREATE TABLE and ALTER SCHEMA permissions to create tables.


# Create Views
- Views are virtual tables that are based on the result set of a SELECT statement.
- A single view can reference one or more tables.
- You can use views as a source for your queries in much the same way as tables.
- However, views don't persistently store data.

> https://learn.microsoft.com/en-us/sql/relational-databases/views/views?view=sql-server-ver16

---

- The view can use for security. For example, you can create a view that shows only specific columns to a user.
- And User can not write your tables directly.
- Views can be considered an application programming interface (API) to a database for purposes of retrieving data.

```tsql
CREATE VIEW <schema_name.view_name> [<column_alias_list>] 
[WITH <view_options>]
AS select_statement;
```

> The ORDER BY clause is not permitted in a view definition unless the view uses a TOP, OFFSET/FETCH, or FOR XML element.

---

Example:

```tsql
CREATE VIEW Sales.CustOrders
AS
SELECT
  O.custid, 
  DATEADD(month, DATEDIFF(month, 0, O.orderdate), 0) AS ordermonth,
  SUM(OD.qty) AS qty
FROM Sales.Orders AS O
  JOIN Sales.OrderDetails AS OD
    ON OD.orderid = O.orderid
GROUP BY custid, DATEADD(month, DATEDIFF(month, 0, O.orderdate), 0);
```

```tsql
SELECT custid, ordermonth, qty
FROM Sales.CustOrders;

SELECT * FROM Sales.CustOrders;
```

## Indexed Views
- This means the view definition has been computed and the resulting data stored just like a table. You index a view by creating a unique clustered index on it.
- It's not usable when our data is frequently changing.

> https://learn.microsoft.com/en-us/sql/relational-databases/views/create-indexed-views?view=sql-server-ver16


# Temporary Tables
- Temporary tables are tables that exist temporarily on the SQL Server.
- Use local temporary tables to create tables scoped to your current session.
- Multiple users can create tables using the same name, and they would have no effect on each other.
- you'd add `#` before the table name to signify that it's a local temporary table:

```tsql
CREATE TABLE #TempTable
(
    ID INT,
    Name VARCHAR(50)
);
```

---

- We can also create global temporary tables that are available to all users and all sessions.
- In this creation, names have to be unique for all users.
- When session ends, the global temporary table is dropped like a local temporary table.
- You'd add `##` before the table name to signify that it's a global temporary table:

Create:

```tsql
CREATE TABLE ##TempTable
(
    ID INT,
    Name VARCHAR(50)
);
```

Insert:

```tsql
INSERT INTO ##TempTable
VALUES (1, 'John');
```

Select:

```tsql
SELECT * FROM ##TempTable;
```

Drop:

```tsql
DROP TABLE ##TempTable;
```


# Use Common Table Expressions (CTEs)
- A common table expression (CTE) can be thought of as a temporary result set that is defined within the execution scope of a single SELECT, INSERT, UPDATE, DELETE, or CREATE VIEW statement.
- It allows you to reference the result set multiple times in the same statement.

Usage:

```tsql
WITH <CTE_name>
    AS (<CTE_definition>) <SELECT_statement>
```

```tsql
WITH CTE_TABLE AS
(
    SELECT
        ID,
        Name
    FROM
        Table
)
SELECT * FROM CTE_TABLE;
```

> CTEs support recursion, in which the expression is defined with a reference to itself.
> https://learn.microsoft.com/en-us/sql/t-sql/queries/with-common-table-expression-transact-sql?view=sql-server-ver16


# Derived Tables
- A derived table is a subquery that can be used in the FROM clause of a SELECT, INSERT, UPDATE, or DELETE statement.
- This select not allow to use ORDER BY clause. We have to sort the data in the outer query.

Usage:

```tsql
SELECT
    *
FROM
    (SELECT
        ID,
        Name
    FROM
        Table) AS DerivedTable;
```

---

We can pass arguments to the derived table:

```tsql
DECLARE @ID INT = 1;
SELECT
    *
FROM
    (SELECT
        ID,
        Name
    FROM
        Table
    WHERE
        ID = @ID) AS DerivedTable;
```






