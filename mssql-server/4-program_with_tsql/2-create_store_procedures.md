# Store Procedures
Stored procedures are named groups of Transact-SQL (T-SQL) statements that can be used and reused whenever they're needed.

W have some advantages of using stored procedures:

- Reusability: You can write a stored procedure once and use it many times.
- Security: You can grant permissions to users to execute a stored procedure without giving them permission to the underlying tables.
- Improve Quality: You can also include appropriate error handling code and make sure that each stored procedure is properly tested before being used in a production environment.
- Improve Performance: When stored procedures are first executed, an execution plan is created. That execution plan can be reused when the stored procedure is executed again. This is typically quicker than creating an execution plan every time the code is executed.
- Lower Maintenance: Stored procedures provide an interface to the data tier. When changes to the underlying database objects change, only the procedures are updated providing a clean separation between the data and application tiers.

---

There are 3 types of stored procedures:
- User-defined stored procedures
- Temporary stored procedures
- System stored procedures

## Call Stored Procedures
When an application or user executes a stored procedure, the EXECUTE command or its shortcut, EXEC is used, followed by the two-part name of the procedure.

```tsql
EXEC dbo.uspGetEmployeeManagers;
```

---

System stored procedures are prefixed with sp_. System stored procedures aren't created by users, but are part of all user-defined and system-defined databases. 
They don't require a fully qualified name to be executed, but it's best practice to include the sys schema name.

`EXEC sys.sp_helpdb;`

### Automatically Executing Stored Procedures
You can configure a stored procedure to execute automatically when SQL Server starts. 
This is useful for procedures that need to run on a regular basis, such as maintenance tasks.

To do this, you can use the `sp_procoption` system stored procedure.

```tsql
EXEC sp_procoption @ProcName = 'uspGetEmployeeManagers', -- The name of the stored procedure
     @OptionName = 'startup', -- The option to set (startup, shutdown, or both)
     @OptionValue = 'on'; -- The value to set the option to (on or off).
```

To execute multiple procedures that don't need to execute them in parallel, make one procedure the startup procedure and call the other procedures from the startup procedure. 
This will use only one worker thread.

Startup procedures must be in the master database.


## Pass Parameters to Stored Procedures
One of the advantages of using stored procedures is that you can pass parameters to them at runtime.
We have 2 types of parameters:
- Input parameters
- Output parameters: If parameter marked as an OUTPUT parameter, the value of the parameter can be changed by the stored procedure.

### Input Parameters
Stored procedures declare their input parameters by name and data type in the header of the CREATE PROCEDURE statement.
Input parameters are the default type of parameter.

Parameter names must be prefixed by the @ character, and be unique in the scope of the procedure.

```tsql
-- Create a stored procedure that accepts an input parameter
CREATE PROCEDURE Products.ProductsBySupplier
    @supplierid INT
AS
SELECT ProductID, ProductName, UnitPrice
FROM Products
WHERE SupplierID = @supplierid;
RETURN;
GO;
    
-- Execute the stored procedure
EXEC Products.ProductsBySupplier @supplierid = 5;
```

---

However, parameters must be passed either by name or by position

```tsql
EXEC Products.ProductsBySupplier 5;
```

---

Check that parameters are of the correct data type. For example, if a procedure accepts an NVARCHAR, pass in the Unicode character string format: **N'string'**.

---

We can see store procedures in the Object Explorer in SSMS under the Programmability > Stored Procedures node.
Also, we can see the parameters of a stored procedure.
We have 3 information about parameters:
- Name
- Data Type
- An **in** or **out** indicator (It's an arrow pointing to the down for input parameters and to the above for output parameters)

You can query a system catalog view such as **sys.parameters** to retrieve parameter definitions together with the object ID.


### Default Values for Parameters
You can assign default values to parameters in a stored procedure.

```tsql
CREATE PROCEDURE Products.ProductsBySupplier
    @supplierid INT = 5
AS
SELECT ProductID, ProductName, UnitPrice
FROM Products
WHERE SupplierID = @supplierid;
RETURN;
GO;

-- Execute the stored procedure
EXEC Products.ProductsBySupplier;
```

### Output Parameters
Output parameters are used to return values from a stored procedure to the calling program.

```tsql
CREATE PROCEDURE Products.GetProductCount
    @productcount INT OUTPUT,
    @listPrice money OUT,
    @supplierid INT = 5
AS
SELECT @productcount = COUNT(*), @listPrice = AVG(ListPrice)
FROM Products
WHERE SupplierID = @supplierid;
RETURN;
GO;

-- Execute the stored procedure
DECLARE @getproductcount INT, @getlistPrice money;
EXEC Products.GetProductCount 
     @productcount = @getproductcount OUTPUT, 
     @listPrice = @getlistPrice OUTPUT, 
     @supplierid = 5;
SELECT @getproductcount, @getlistPrice;
```