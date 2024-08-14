# Subqueries
- A subquery is a SELECT statement nested, or embedded, within another query.
- The purpose of a subquery is to return results to the outer query. 
- The form of the results will determine whether the subquery is a scalar or multi-valued subquery:
  - Scalar subquery: Returns a single value (Single-row, single-column result set).
  - Multi-valued subquery: Returns multiple values (Single-column, multi-row result set).

---


## Scaler Queries And Multi-valued Queries

- Scaler query example:
- We can use scaler subquery in the where clause, select list, having clause, from, even update and delete statements.

```sql
-- EXAMPLE 1
-- This query returns the maximum value of the SalesOrderID column from the Sales.SalesOrderHeader table.
SELECT MAX(SalesOrderID)
FROM Sales.SalesOrderHeader;


-- EXAMPLE 2
SELECT SalesOrderID, ProductID, OrderQty
FROM Sales.SalesOrderDetail
WHERE SalesOrderID = (
        SELECT MAX(SalesOrderID)
        FROM Sales.SalesOrderHeader
       );


-- EXAMPLE 3
SELECT SalesOrderID, ProductID, OrderQty,
        (
            SELECT AVG(OrderQty)
            FROM SalesLT.SalesOrderDetail
        ) AS AvgQty
FROM SalesLT.SalesOrderDetail
WHERE SalesOrderID =(
      SELECT MAX(SalesOrderID)
      FROM Sales.SalesOrderHeader
     );

```

- Multi-valued query example:
- We can use multi-valued subquery in the where clause, having clause, from.

```sql
-- EXAMPLE 1
-- This query returns the SalesOrderID and CustomerID columns from the Sales.SalesOrderHeader table for all customers in Canada.
SELECT CustomerID, SalesOrderID
FROM Sales.SalesOrderHeader
WHERE CustomerID IN (
  SELECT CustomerID
  FROM Sales.Customer
  WHERE CountryRegion = 'Canada');

-- EXAMPLE 2
-- This query is equivalent to the previous query, but it uses a JOIN clause instead of a subquery.
SELECT c.CustomerID, o.SalesOrderID
FROM Sales.Customer AS c
       JOIN Sales.SalesOrderHeader AS o
            ON c.CustomerID = o.CustomerID
WHERE c.CountryRegion = 'Canada';




-- EXAMPLE 3
-- Multi-valued subquery in the FROM clause
SELECT c.CustomerID, o.SalesOrderID
FROM Sales.Customer AS c
       JOIN (
            SELECT CustomerID, SalesOrderID
            FROM Sales.SalesOrderHeader
            WHERE OrderDate BETWEEN '20060701' AND '20060731'
       ) AS o
       ON c.CustomerID = o.CustomerID
WHERE c.CountryRegion = 'Canada';


-- EXAMPLE 4
-- This query is equivalent to the previous query, but it uses a JOIN clause instead of a subquery.
SELECT c.CustomerID, o.SalesOrderID
FROM Sales.Customer AS c
       JOIN Sales.SalesOrderHeader AS o
            ON c.CustomerID = o.CustomerID
WHERE c.CountryRegion = 'Canada' AND o.OrderDate BETWEEN '20060701' AND '20060731';
```

- So how do you decide whether to write a query involving multiple tables as a JOIN or with a subquery? 
- Sometimes, it just depends on what you’re more comfortable with. 
- Most nested queries that are easily converted to JOINs will actually BE converted to a JOIN internally. 
- For such queries, there is then no real difference in writing the query one way vs another.
- If we need to columns from both tables, then JOIN is the best option.
- If we need to filter the rows from the outer query based on the result of the inner query, then subquery is the best option.


## Self-contained Subqueries And Correlated Subqueries
- **Self-contained subquery:** A subquery that can run independently of the outer query.
- **Correlated subquery:** A subquery that depends on the outer query for its values.

```sql
SELECT SalesOrderID, CustomerID, OrderDate
FROM SalesLT.SalesOrderHeader AS o1
WHERE SalesOrderID =
    (SELECT MAX(SalesOrderID)
     FROM SalesLT.SalesOrderHeader AS o2
     WHERE o2.CustomerID = o1.CustomerID)
ORDER BY CustomerID, OrderDate;
```

- Correlated subqueries cannot be executed separately from the outer query. This restriction complicates testing and debugging.
- Unlike self-contained subqueries, which are processed once, correlated subqueries will run multiple times. 
- Logically, the outer query runs first, and for each row returned, the inner query is processed.


## Working With Exists And Not Exists
- The EXISTS operator is used to test for the existence of any rows in a subquery.
- It's working with a correlated subquery.
- The EXISTS operator returns TRUE if the subquery returns one or more rows.

- When we use the EXISTS operator, sql server can understand that we are only interested in the existence of rows.
- It's prepare the execution plan accordingly.
- But subquery will return all the rows too. Because it's a correlated subquery. 

Conceptually, the EXISTS operator works as follows:
```sql
SELECT CustomerID, CompanyName, EmailAddress 
FROM Sales.Customer AS c 
WHERE
(SELECT COUNT(*) 
  FROM Sales.SalesOrderHeader AS o
  WHERE o.CustomerID = c.CustomerID) > 0;


-- The same query can be written using the EXISTS operator as follows:
SELECT CustomerID, CompanyName, EmailAddress
FROM Sales.Customer AS c
WHERE EXISTS
        (SELECT *
         FROM Sales.SalesOrderHeader AS o
         WHERE o.CustomerID = c.CustomerID);
```


# References
https://learn.microsoft.com/en-us/training/modules/write-subqueries/