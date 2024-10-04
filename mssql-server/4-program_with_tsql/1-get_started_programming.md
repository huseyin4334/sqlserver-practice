# Introduction
Transact-SQL (T-SQL) provides a robust programming language with features that let you temporarily store values in variables, apply conditional execution of commands, pass parameters to stored procedures, and control the flow of your programs.


## Description
Transact-SQL (T-SQL) is a proprietary extension of the open standard Structured Query Language (SQL). 
It supports declared variables, string and data processing, error and exception handling, and transaction control. 
While SQL is a programming language, T-SQL adds support for procedural programming and the use of local variables.

A T-SQL program will usually start with a `BEGIN` statement and end with an `END` statement.

- IF..ELSE - A conditional statement that lets you decide what aspects of your code will execute.
- WHILE - A looping statement that is ideal for running iterations of T-SQL statements.
- DECLARE - You'll use this to define variables.
- SET - One of the ways you'll assign values to your variables.
- BATCHES - Series of T-SQL statements that are executed as a unit.


# Describe Batches
T-SQL batches are a group of one or more Transact-SQL statements sent to SQL Server for execution as a single unit.

How you mark the end of a batch depends on the settings of your client. 
Clients like SQL Server Management Studio (SSMS) and SQLCMD use the `GO` keyword to mark the end of a batch.


Syntax:
```tsql
CREATE NEW <view_name>
AS ....
GO
CREATE PROCEDURE <procedure_name>
AS ....
GO
```

---
The batch terminator `GO` isn't a Transact-SQL statement. 
But this keyword is recognised by the SQLCMD and SQL Server Management Studio (SSMS) tools to indicate the end of a batch of Transact-SQL statements to be sent to SQL Server for execution.


- Keep in mind;
  - Batches are boundaries for variable scope. Which means a variable declared in one batch is not available in another batch.
  - Some statements, like `CREATE VIEW` and `CREATE PROCEDURE`, must be the first statement in a batch. May not be combined with other statements in the same batch.


---

When a batch submitted by a client (click Execute in SSMS);
- The batch is parsed for syntax errors by the SQL Server Engine.
  - If there are syntax errors, the batch is rejected (all statements), and the client receives an error message.
- If the batch is accepted, the SQL server runs other steps, resolving object names, checking permissions, and optimizing the query.
- When processes are complete, the SQL Server Engine starts execution of the batch.
- When batch starts the executing every error will be runtime error.
  - When we get an error in 1 line of the batch, the execution of the batch will stop. The error message will be displayed in the client tool.


# Declare And Assign Variables And Synonyms
In T-SQL, as with other programming languages, you can declare variables to store data temporarily.


## Variables

In T-SQL, variables must be declared before they can be used.
They may be assigned a value or initialized at the time of declaration.

We should use the `DECLARE` statement to declare a variable.

Syntax:
```tsql
DECLARE @variable_name data_type;

DECLARE @variable_name data_type = value;


-- Example
DECLARE @variable_name INT = 10;

DECLARE @variable_name VARCHAR(50) = 'Hello World';
```

Variables scope is limited to the batch in which they are declared. A variable is automatically destroyed when the batch ends.

---

Once you've declared a variable, you must initialize it, or assign it a value. You can do that in three ways:
- In Sql server 2008 and later versions, you can assign a value to a variable at the time of declaration.
- You can assign a value to a variable using the `SET` statement.
- You can assign a value to a variable using a `SELECT` statement.

Example:
```tsql
DECLARE @var1 INT = 10;
DECLARE @var2 NVARCHAR(50);
SET @var2 = N'Hello World'; -- N is used to specify the Unicode string.

DECLARE @var3 NVARCHAR(20);
SELECT @var3 = lastName FROM Sales.Customer WHERE customerId = 1;

select @var1, @var2, @var3;
GO
```

| @var1 | @var2       | @var3 |
|-------|-------------|-------|
| 10    | Hello World | Dav   |


> For more: [DECLARE (Transact-SQL)](https://learn.microsoft.com/en-us/sql/t-sql/language-elements/variables-transact-sql?view=sql-server-ver16)


## Synonyms
In sql server, a synonym is an alias or alternative name for a table, view, sequence, or other schema-bound object.
Synonyms can be used to reference objects in other databases or on other servers.
For example, you can create a synonym for a table in another database, and then reference that synonym in a query.
We have 3 commands to work with synonyms. `CREATE SYNONYM`, `DROP SYNONYM`, and `ALTER SYNONYM`.

Example:
```tsql
CREATE SYNONYM dbo.MySynonym FOR AdventureWorks2012.Sales.Customer;
GO;
EXEC dbo.MySynonym @numrows = 10, @catid = 1;
```

To create a synonym, you must have 'CREATE SYNONYM' permission as well as permission to alter the schema in which the synonym will be stored.


# If And While Blocks
These elements include, but aren't limited to:
- IF...ELSE, which executes code based on a Boolean expression.
- WHILE, which creates a loop that executes providing a condition is true.
- BEGIN…END, which defines a series of T-SQL statements that should be executed together.
- Other keywords, for example, BREAK, CONTINUE, WAITFOR, and RETURN, which are used to support T-SQL control-of-flow operations.


When if block have False or Unknown condition, the block will not be executed.

Example:
```tsql
IF OBJECT_ID('dbo.tl') IS NOT NULL
    DROP TABLE dbo.tl
GO
```

```tsql
USE AdventureWorks2012;
GO;
IF OBJECT_ID('dbo.tl') IS NULL
BEGIN
  PRINT 'Creating table tl';
  CREATE TABLE dbo.tl (c1 INT);
END
ELSE
BEGIN
  PRINT 'Dropping table tl';
  DROP TABLE dbo.tl;
END
GO
```

```tsql
IF EXISTS (SELECT * FROM Sales.EmpOrders WHERE empid =5) -- Check if the employee has associated orders
BEGIN
    PRINT 'Employee has associated orders';
END;
```

```tsql
DECLARE @empid INT = 5, @name NVARCHAR(50);

WHILE @empid <= 10
  BEGIN
      SELECT @name = name FROM Sales.Employee WHERE empid = @empid;
      PRINT @name;
      SET @empid = @empid + 1;
      IF @empid = 8
          BREAK;
      IF @empid = 3
          CONTINUE;
  END;
```
