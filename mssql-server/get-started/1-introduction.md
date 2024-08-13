# The Sql Language
- There is a SQL language standard defined by the American National Standards Institute (ANSI). Each vendor adds their own variations and extensions.
- Relational Database Management Systems (RDMS) like MS SQL Server, etc. use SQL as their standard database language.
- Sql is a declarative language, which means that you specify what you want to do, not how you want to do it.
- You use SQL to describe the results you want, and the database engine's query processor develops a query plan to retrieve it. The query processor uses statistics about the data in the database and indexes that are defined on the tables to come up with a good query plan.


# Transact-SQL
T-SQL is **dialect** of SQL that is used by Microsoft SQL Server.
Basic SQL statements, such as SELECT, INSERT, UPDATE, and DELETE are available no matter what relational database system you're working with.
Although these SQL statements are part of the ANSI SQL standard, many database management systems also have their own extensions. 
These extensions provide functionality not covered by the SQL standard, and include areas such as security management and programmability. 
Microsoft database systems such as SQL Server, Azure SQL Database, Azure Synapse Analytics, and others use a dialect of SQL called Transact-SQL, or T-SQL. 
T-SQL includes language extensions for writing stored procedures and functions, which are application code that is stored in the database, and managing user accounts.
Briefly, T-SQL is a superset of the ANSI SQL standard, and includes additional features that are specific to Microsoft's SQL Server.

> Dialect: https://medium.com/@abhapratiti27/understanding-sql-dialects-a-deeper-dive-into-the-linguistic-variations-of-sql-e7e2fdb7509b#:~:text=SQL%20dialects%20are%20essentially%20different,syntax%2C%20functions%2C%20and%20capabilities.


# Set Based Processing
- Set is a collection of objects. For instance, Customer table is a set of customers.
- It is important to always think of the entire set, instead of individual members. This mindset will better equip you to write set-based code, instead of thinking one row at a time.
- In the set theory, we can access the row in any order. If we need the order of the rows, we need to use the ORDER BY clause.


# Schema
- A schema is a collection of database objects, including tables, views, indexes, and stored procedures.
- A schema is namespace for database objects. It is a way to group objects together.
- Sql server use a hierarchical naming system to identify objects. The naming system is `schema.object`. For example, `dbo.Customer` refers to the Customer table in the dbo schema.
- If we refer to an object in another server and database, we use the four-part naming system: `server.database.schema.object`.


# Explore The Structure of Sql Statements
- SQL statements are made up of clauses, which are keywords that define the purpose of the process.
- Generally, statements grouping in 3 categories:
    - **Data Manipulation Language (DML) statements:**
      - (DML) is the set of SQL statements that focuses on querying and modifying data.
      - SELECT, INSERT, UPDATE, DELETE
    - **Data Definition Language (DDL) statements:**
      - (DDL) is the set of SQL statements that handles the definition and life cycle of database objects, such as tables, views, and procedures.
      - CREATE, ALTER, DROP
    - Data Control Language (DCL) statements: 
      - (DCL) is the set of SQL statements used to manage security permissions for users and objects.
      - GRANT, REVOKE, DENY
> Note: Sometimes you may also see **TCL** listed as a type of statement, to refer to Transaction Control Language. 
> In addition, some lists may redefine **DML** as **Data Modification Language**, which wouldn't include SELECT statements, but then they add **DQL** as **Data Query Language** for SELECT statements.

- DML statements are also used by application developers to perform "**CRUD**" operations to create, read, update, or delete application data.


# Data Types
- Data types are used to define the type of data that can be stored in a column.
- Every column and variables used in T-SQL each have a data type.

| Exact Numeric   | Approximate Numeric | Character | Date and Time  | Binary    | Other            |
|-----------------|---------------------|-----------|----------------|-----------|------------------|
| tinyint         | float               | char      | date           | binary    | cursor           |
| smallint        | real                | varchar   | time           | varbinary | hiearchyid       |
| int             |                     | text      | datetime       | image     | sql_variant      |
| bigint          |                     | nchar     | datetime2      | timestamp | table            |
| bit             |                     | nvarchar  | smalldatetime  |           | timestamp        |
| decimal/numeric |                     | ntext     | datetimeoffset |           | uniqueidentifier |
| numeric         |                     |           |                |           | xml              |
| money           |                     |           |                |           | geography        |
| smallmoney      |                     |           |                |           | geometry         |

> Details: https://learn.microsoft.com/en-us/sql/t-sql/data-types/data-types-transact-sql?view=sql-server-ver16

## Conversions
- Converting data types is a common task when working with SQL Server.
- The supported data type conversions are defined by the database API.
- Conversion Types:
- Explicit Conversions
    - Explicit conversions use the **CAST** or **CONVERT** functions.
- Implicit Conversions
    - Implicit conversions are not visible to the user.
    - SQL Server automatically converts the data from one data type to another.
    - For example, when a **smallint** is compared to an **int**, the **smallint** is implicitly converted to **int** before the comparison proceeds.
- We can concatenate data types using the + operator.
- We can use + and - operators to perform arithmetic operations on numeric data types.

> Details: https://learn.microsoft.com/en-us/sql/t-sql/data-types/data-type-conversion-database-engine?view=sql-server-ver16
> Cast And Convert: https://learn.microsoft.com/en-us/sql/t-sql/functions/cast-and-convert-transact-sql?view=sql-server-ver16

### CAST And TRY_CAST
- The CAST function converts a value to a specified data type if the value is compatible with the target data type. An error will be returned if incompatible.
- The TRY_CAST function is similar to CAST, but it returns a NULL value if the conversion fails.

```tsql
-- Cast will convert the ProductID column to a varchar data type.
-- The ProductID column is an integer data type. If we try to concatenate it without cast with a string, we will get an error.
SELECT CAST(ProductID AS varchar(4)) + ': ' + Name AS ProductName
FROM Production.Product;

-- CAST and TRY_CAST functions can be used to convert data types. 
-- But if the conversion fails, CAST will return an error, while TRY_CAST will return a NULL value.
SELECT CAST(Size AS integer) As NumericSize
FROM Production.Product;

-- Error: Conversion failed when converting the nvarchar value 'M' to data type int.
-- Text based data types can be 'S', 'M', 'L', etc. But we can't convert them to integer. 


SELECT TRY_CAST(Size AS integer) As NumericSize
FROM Production.Product;
```


### CONVERT And TRY_CONVERT
- CAST is the ANSI standard SQL function for converting between data types, and is used in many database systems. 
- In Transact-SQL, you can also use the CONVERT function, as shown here:
- Another benefit of using CONVERT over CAST is that CONVERT also includes a parameter that enables you specify a format style when converting numeric and date values to strings.

```tsql

SELECT SellStartDate,
       CONVERT(varchar(20), SellStartDate) AS StartDate,
       -- The 101 style code specifies the format as mm/dd/yyyy.
       CONVERT(varchar(10), SellStartDate, 101) AS FormattedStartDate
FROM SalesLT.Product;
```

| SellStartDate               | StartDate          | FormattedStartDate |
|-----------------------------|--------------------|--------------------|
| 2011-05-31T00:00:00.0000000 | 2011-05-31 12:00AM | 05/31/2011         |

> Styles: https://learn.microsoft.com/en-us/sql/t-sql/functions/cast-and-convert-transact-sql?view=sql-server-ver16#date-and-time-styles

> When we convert decimal or numeric data types to each other, we should know precision and scale values. 
> Details: https://learn.microsoft.com/en-us/sql/t-sql/data-types/decimal-and-numeric-transact-sql?view=sql-server-ver16


### PARSE And TRY_PARSE
- The PARSE function is designed to convert formatted strings that represent numeric or date/time values.

```tsql
SELECT PARSE('01/01/2021' AS date) AS DateValue,
       PARSE('$199.99' AS money) AS MoneyValue;
```

| DateValue  | MoneyValue |
|------------|------------|
| 2021-01-01 | 199.99     |


### STR
- The STR function converts a numeric value to a varchar.
- If the length isn't big enough to hold the decimal value, the function will return asterisks (**).
- `STR ( float_expression [ , length [ , decimal ] ] )`

```tsql
SELECT '$' + STR(123.4167, 6, 1) AS DecimalValue,
       '$' + STR(123.4967, 6, 2) AS DecimalValue2,
       '$' + STR(123.4967, 2, 2) AS DecimalValue3;



-- | DecimalValue | DecimalValue2 | DecimalValue3 |
-- |--------------|---------------|---------------|
-- | $123.4       | $123.5        | $**           |
```


# Precision and Scale
- Precision is the total number of digits in a number (including the digits to the left and right of the decimal point).
- Scale is the number of digits to the right of the decimal point in a number.
- Precision can be from 1 to 38, and scale can be from 0 to the precision value.
- The ISO synonyms for **decimal** are **dec** and **dec(p,s)**. **numeric** is functionally **identical** to **decimal**.

```tsql

CREATE TABLE dbo.MyTable (
    -- The DECIMAL data type has a precision of 5 and a scale of 2.
    DecimalCol DECIMAL(5,2),
    -- The NUMERIC data type has a precision of 10 and a scale of 5.
    NumericCol NUMERIC(10,5)
);

INSERT INTO dbo.MyTable values (123, 12345.12);

SELECT DECIMALCOL, NUMERICCOL FROM dbo.MyTable;

-- MyDecimalColumn  MyNumericColumn
---------------- ----------------
-- 123.00           12345.12000
```


# Handle Nulls
- A NULL value means no value or unknown.

## ISNULL
- The ISNULL function replaces NULL with the specified replacement value.
- Replace value have to be the same data type with the column.

```tsql
SELECT ISNULL(FirstName, 'No First Name') AS FirstName,
         ISNULL(Price, 0) AS Price
FROM Person.Person;

-- | FirstName      | Price |
-- |----------------| ------|
-- | No First Name  | 0     |
-- | Ken            | 234.56|
```


## COALESCE
- The COALESCE function returns the first non-NULL expression among its arguments.
- The ISNULL function is not ANSI standard, but the COALESCE function is.
- COALESCE can take multiple arguments, while ISNULL can only take two.
- `SELECT COALESCE ( expression1, expression2, [ ,...n ] )`

```tsql
-- The important thing to remember is that COALESCE returns the first non-NULL value in the list.
-- In HR.Wages table, HourlyRate, WeeklySalary, and Commission columns just one of them has a value.
-- And we want to get the WeeklyEarnings column that has the value of the first non-NULL column.
SELECT EmployeeID,
      COALESCE(HourlyRate * 40,
                WeeklySalary,
                Commission * SalesQty) AS WeeklyEarnings
FROM HR.Wages;

```


## NULLIF
- NULLIF returns NULL if the two expressions are equal.
- It useful when you want to clean the data.
- We can clean blank or placeholder values with NULLIF.
- `SELECT NULLIF ( expression1, expression2 )`
  - expression1: The first expression to compare with the second expression. If the two expressions are equal, NULLIF returns a NULL value.
  - expression2: The second expression to use for comparison.

```tsql
-- If the FirstName column is an empty string, NULLIF will return NULL.
-- If the Price column is 0, NULLIF will return NULL.
SELECT NULLIF(FirstName, '') AS FirstName,
       NULLIF(Price, 0) AS Price
FROM Person.Person;

```


## Statements

## SELECT Statement
- The SELECT statement is used to retrieve data from a database.

```tsql
-- COUNT(OrderID) is an aggregate function that counts the number of rows in the result set.
-- AS Orders is an alias that gives the COUNT(OrderID) column a more descriptive name.
SELECT OrderDate, COUNT(OrderID) AS Orders
-- FROM Sales.SalesOrder specifies the table from which to retrieve data.
FROM Sales.SalesOrder
-- WHERE Status = 'Shipped' specifies the condition that must be met for a row to be included in the result set.
WHERE Status = 'Shipped'
-- GROUP BY OrderDate groups the result set by the OrderDate column.
GROUP BY OrderDate
-- HAVING COUNT(OrderID) > 1 specifies the condition that must be met for a group to be included in the result set after the GROUP BY clause has been applied.
HAVING COUNT(OrderID) > 1
-- ORDER BY OrderDate DESC specifies the order in which the rows are returned. (Descending or Ascending)
ORDER BY OrderDate DESC;
```

Now that you've seen what each clause does, let's look at the order in which SQL Server actually evaluates them;
1. The **FROM** clause is evaluated first, to provide the source rows for the rest of the statement. A virtual table is created and passed to the next step.
2. The **WHERE** clause is next to be evaluated, filtering those rows from the source table that match a predicate. The filtered virtual table is passed to the next step.
3. **GROUP BY** is next, organizing the rows in the virtual table according to unique values found in the GROUP BY list. A new virtual table is created, containing the list of groups, and is passed to the next step. From this point in the flow of operations, only columns in the GROUP BY list or aggregate functions may be referenced by other elements.
4. The **HAVING** clause is evaluated next, filtering out entire groups based on its predicate. The virtual table created in step 3 is filtered and passed to the next step.
5. The **SELECT** clause finally executes, determining which columns will appear in the query results. Because the SELECT clause is evaluated after the other steps, any column aliases (in our example, Orders) created there cannot be used in the GROUP BY or HAVING clause.
6. The **ORDER BY** clause is the last to execute, sorting the rows as determined by its column list.

As you have seen, The order of the clauses don't same with the logical orders.
Logical order made by the SQL Server Query Processor to optimize the query.

### Expressions
- An expression is a combination of one or more values, operators, and SQL functions that evaluates to a value.
- Expressions can be used in the SELECT list, WHERE clause, HAVING clause, ORDER BY clause, and other parts of a SQL statement.

```tsql
SELECT ProductID,
       -- This expression concatenates the Name and ProductNumber columns.
       Name + '(' + ProductNumber + ')' AS ProductName,
       -- This expression calculates the difference between ListPrice and StandardCost.
       ListPrice - StandardCost AS PriceDifference
FROM Production.Product;
```

| ProductID | ProductName                           | PriceDifference |
|-----------|---------------------------------------|-----------------|
| 680       | HL Road Frame - Black, 58(FR-R92B-58) | 372.19          |

---


# References
https://learn.microsoft.com/en-us/training/modules/introduction-to-transact-sql/