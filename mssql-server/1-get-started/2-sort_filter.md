# Sort Results
- In the logical order of query processing, ORDER BY is the last phase of a SELECT statement to be executed.
- SQL Server doesn't guarantee the physical order of rows in a table.

```tsql
    SELECT <select_list>
    FROM <table_source>
    ORDER BY <order_by_list> [ASC|DESC];
```

- ASC is the default sort order. ASC is ascending, and DESC is descending.
- ORDER BY list can take several types of elements in its list:
    - **Columns by name** in the SELECT list.
    - **Aliases** used in the SELECT list.
    - **Columns by ordinal position** in the SELECT list.
      - It's not recommended to use ordinal positions in the ORDER BY list because of readability and maintainability.
    - **Columns not included in the SELECT list, but available from tables listed in the FROM clause.**
      - If the query uses a DISTINCT option, any columns in the ORDER BY list must be included in the SELECT list.

    
```tsql
-- Alias in ORDER BY list
-- Columns not included in the SELECT list, but available from tables listed in the FROM clause.
    SELECT ProductCategoryID AS Category, ProductName
    FROM Production.Product
    ORDER BY Category ASC, Price DESC;
```


# Limit Results (TOP)
- The TOP clause is a Microsoft-proprietary extension of the SELECT clause.
- The TOP works after the ORDER BY clause.

```tsql
    SELECT TOP (expression) [PERCENT] [WITH TIES] <select_list>
    FROM <table_source>
    ORDER BY <order_by_list>;
```

## With Ties
- The WITH TIES option includes rows that match the last row in the result set.
- The WITH TIES option have to be used with the TOP clause and ORDER BY clause.

```tsql
    SELECT TOP 2 WITH TIES Name, ListPrice
    FROM Production.Product
    ORDER BY ListPrice DESC;
```

| Name                      | ListPrice |
|---------------------------|-----------|
| HL Road Frame - Black, 58 | 1831.50   |
| HL Road Frame - Red, 58   | 1431.50   |
| HL Road Frame - Red, 62   | 1431.50   |
| HL Road Frame - Red, 44   | 1431.50   |

- We limited the result to 2 rows, but the WITH TIES option included the rows that matched the last row in the result set.


## PERCENT
- The PERCENT option specifies the number of rows or the percentage of rows to return.
- The PERCENT option have to be used with the TOP clause.
- For the purposes of row count, TOP (N) PERCENT will round up to the nearest integer.

```tsql
    SELECT TOP 2 PERCENT Name, ListPrice
    FROM Production.Product
    ORDER BY ListPrice DESC;
```

| Name                      | ListPrice |
|---------------------------|-----------|
| HL Road Frame - Black, 58 | 1831.50   |
| HL Road Frame - Red, 58   | 1431.50   |
| .....                     | ....      |
(11 rows affected)

- We limited the result to 2 percent of the rows in the result set.
- Total rows in the result set: 504
- 2 percent of 504 is 10.08, so the result set includes 11 rows.


# Page Results (OFFSET FETCH)
- The OFFSET FETCH clause is a Microsoft-proprietary extension of the SELECT clause.
- OFFSET and FETCH are used to skip and return a specified number of rows from a result set.
- The OFFSET FETCH clause works after the ORDER BY clause.
- The OFFSET specifies the number of rows to skip. (Basicly, it's the starting point.)
- The FETCH specifies the number of rows to return.
- OFFSET is required, but FETCH is optional.

```tsql

-- OFFSET { integer_constant | offset_row_count_expression } { ROW | ROWS }
-- [FETCH { FIRST | NEXT } {integer_constant | fetch_row_count_expression } { ROW | ROWS } ONLY]

    SELECT <select_list>
    FROM <table_source>
    ORDER BY <order_by_list>
    OFFSET <offset_expression> ROWS
    FETCH NEXT <fetch_expression> ROWS ONLY;
```

> **Note:** There's no server-side state maintained, and you'll need to track your position through a result set via client-side code.

## Examples
- Get 2 page with 10 rows per page.
```tsql
    SELECT ProductID, Name, ListPrice
    FROM Production.Product
    ORDER BY ListPrice DESC
    OFFSET 0 ROWS -- Skip the first 0 rows
    FETCH NEXT 10 ROWS ONLY; -- Return the next 10 rows
    
    SELECT ProductID, Name, ListPrice
    FROM Production.Product
    ORDER BY ListPrice DESC
    OFFSET 10 ROWS -- Skip the first 10 rows
    FETCH NEXT 10 ROWS ONLY; -- Return the next 10 rows
```


# REMOVE DUBLICATES (DISTINCT)
- By default, the SELECT statement includes an implicit ALL keyword.

```tsql
    SELECT City, CountryRegion
    FROM Production.Supplier
    ORDER BY CountryRegion, City;
    
    /*
        SELECT ALL City, CountryRegion
        FROM Production.Supplier
        ORDER BY CountryRegion, City;
     */
```

| City   | CountryRegion |
|--------|---------------|
| Aachen | Germany       |
| Aachen | Germany       |
| Aachen | Germany       |
| Barrie | Canada        |

---

- To remove duplicates, use the DISTINCT keyword.

```tsql
    SELECT DISTINCT City, CountryRegion
    FROM Production.Supplier
    ORDER BY CountryRegion, City;
```

| City   | CountryRegion |
|--------|---------------|
| Aachen | Germany       |
| Barrie | Canada        |


# Filter Results (WHERE)
- `=` (equals)
  - `PRODUCTID = 680` 
- `<>` (not equals)
  - `PRODUCTID <> 680`
- `>` (greater than)
  - `LISTPRICE > 1000`
- `>=` (greater than or equal to)
  - `LISTPRICE >= 1000`
- `<` (less than)
  - `LISTPRICE < 1000`
- `<=` (less than or equal to)
  - `LISTPRICE <= 1000`
- `BETWEEN` (between an inclusive range)
  - `LISTPRICE BETWEEN 1000 AND 2000`
  - It's inclusive, so it includes the values 1000 and 2000.
- `IN` (matches any value in a list)
  - `PRODUCTID IN (680, 706, 707)`
- `LIKE` (matches a pattern)
  - Characters: `_` (single character), `%` (zero or more characters)
  - `PRODUCTNAME LIKE 'HL%'`
- `IS NULL` (is a null value)
  - `DISCONTINUEDDATE IS NULL`
- `IS NOT NULL` (is not a null value)
  - `DISCONTINUEDDATE IS NOT NULL`
- `AND` (logical AND)
  - `LISTPRICE > 1000 AND LISTPRICE < 2000`
- `OR` (logical OR)
  - `LISTPRICE < 1000 OR LISTPRICE > 2000`
- `NOT` (logical NOT)
  - `NOT LISTPRICE > 1000`
- `EXISTS` (subquery returns any rows)
  - `EXISTS (SELECT * FROM Production.Product WHERE PRODUCTID = 680)`
- `ALL` (compares a value to every value from the subquery)
  - `LISTPRICE > ALL (SELECT LISTPRICE FROM Production.Product WHERE PRODUCTID IN (680, 706, 707))`
- `ANY` (compares a value to any value from the subquery)
  - `LISTPRICE > ANY (SELECT LISTPRICE FROM Production.Product WHERE PRODUCTID IN (680, 706, 707))`


# References
https://learn.microsoft.com/en-us/training/modules/sort-filter-queries/
