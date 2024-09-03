# Union Operator
- The UNION operator is used to combine the result-set of two or more SELECT statements.
  - UNION: The combined result does not include duplicates.
  - UNION ALL: The combined result includes duplicates.

- We have 2 rules:
  - The number and the order of the columns must be the same in all queries.
  - The data types must be compatible.
- If the data types are not compatible or the number of columns is different, we will get an error.

- If you require sorted output, add an ORDER BY clause at the end of the second query.

- When columns have NULL values, the UNION operator treats NULL values as equal.


> UNION is different from JOIN. JOIN compares columns from two tables, to create a result set containing rows from two tables. 
> UNION concatenates two result sets together: all the rows in the first result set are appended to the rows in the second result set.

---

Example:

```tsql
SELECT customerid, companyname, FirstName + ' ' + LastName AS 'Name'
FROM saleslt.Customer
WHERE customerid BETWEEN 1 AND 9
UNION
SELECT customerid, companyname, FirstName + ' ' + LastName AS 'Name'
FROM saleslt.Customer
WHERE customerid BETWEEN 10 AND 19;
```


# Intersect And Except Operators
- The INTERSECT operator returns the common rows between two queries.
- The EXCEPT operator returns the rows that are in the first query but not in the second query.

- We have 2 rules:
  - The number and the order of the columns must be the same in all queries.
  - The data types must be compatible.
- If the data types are not compatible or the number of columns is different, we will get an error.

- If you require sorted output, add an ORDER BY clause at the end of the second query.

---

Example:

First Where Clause Colors:

| ProductID | Color  |
|-----------|--------|
| 500       | Black  |
| 501       | Blue   |
| 502       | Red    |
| 503       | Green  |
| 504       | Yellow |

Second Where Clause Colors:

| ProductID | Color  |
|-----------|--------|
| 751       | Gray   |
| 752       | Blue   |
| 753       | Red    |
| 754       | Green  |
| 755       | Yellow |

```tsql


```tsql
SELECT color FROM SalesLT.Product
WHERE ProductID BETWEEN 500 and 750
INTERSECT
SELECT color FROM SalesLT.Product
WHERE ProductID BETWEEN 751 and 1000;
```

| Color  |
|--------|
| Blue   |
| Red    |
| Green  |
| Yellow |


```tsql
SELECT color FROM SalesLT.Product
WHERE ProductID BETWEEN 500 and 750
EXCEPT
SELECT color FROM SalesLT.Product
WHERE ProductID BETWEEN 751 and 1000;
```

| Color  |
|--------|
| Black  |


```tsql
SELECT color FROM SalesLT.Product
WHERE ProductID BETWEEN 751 and 1000
EXCEPT
SELECT color FROM SalesLT.Product
WHERE ProductID BETWEEN 500 and 750;
```

| Color  |
|--------|
| Gray   |


# APPLY Operator
- The APPLY operator enables queries that evaluate rows in one input set against the expression that defines the second input set.
- The APPLY operator is useful for joining a table with a table-valued function.
- APPLY is more like a JOIN, rather than as a set operator that operates on two compatible result sets of queries.

- We have 2 types of APPLY operators:
  - CROSS APPLY: It returns only the rows that produce a result from the right table.
  - OUTER APPLY: It returns all rows from the left table and the matched rows from the right table.

- The APPLY operator is useful for joining a table with a table-valued function.
- Table-valued functions are functions that return a table as output.

---

Syntax:

```tsql
SELECT <column_list>
FROM left_table_source { CROSS | OUTER } APPLY right_table_source
```

---

Let's start to explain it with INNER JOIN:

```tsql
SELECT oh.SalesOrderID, oh.OrderDate,od.ProductID, od.UnitPrice, od.Orderqty 
FROM SalesLT.SalesOrderHeader AS oh 
INNER JOIN SalesLT.SalesOrderDetail AS od 
ON oh.SalesOrderID = od.SalesOrderID;
```

Now, we will convert it to CROSS APPLY:

```tsql
SELECT oh.SalesOrderID, oh.OrderDate,
od.ProductID, od.UnitPrice, od.Orderqty 
FROM SalesLT.SalesOrderHeader AS oh 
CROSS APPLY (SELECT productid, unitprice, Orderqty 
        FROM SalesLT.SalesOrderDetail AS od 
        WHERE oh.SalesOrderID = od.SalesOrderID
              ) AS od;
```

---

The main difference between these operands is that the right_table_source can use a table-valued function that takes a column from the left_table_source as one of the arguments of the function.




# Summary
> https://learn.microsoft.com/en-us/sql/t-sql/language-elements/operators-transact-sql?view=sql-server-ver16