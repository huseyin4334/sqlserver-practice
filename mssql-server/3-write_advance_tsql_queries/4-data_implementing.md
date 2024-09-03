# Pivot And Unpivot
- **Pivot**: It is used to rotate the table-value by converting the unique values from one column into multiple columns.
- **Unpivot**: It is used to rotate the table-value by converting the multiple columns into multiple rows.
- We can use `Pivot` and `Unpivot` to rotate the table-value.
- To use PIVOT, you need to supply three elements to the operator:
  - **Grouping**: in the FROM clause, you provide the input columns. From those columns, PIVOT will determine which column(s) will be used to group the data for aggregation.
  - **Spreading**: you provide a comma-delimited list of values to be used as the column headings for the pivoted data. The values need to occur in the source data.
  - **Aggregating**: you provide an aggregation function (SUM, and so on) to be performed on the grouped rows.

---

We have a table `Sales` with the following data.

| Category | Qty | Year |
|----------|-----|------|
| A        | 10  | 2019 |
| B        | 20  | 2019 |
| A        | 30  | 2019 |
| C        | 40  | 2019 |
| B        | 50  | 2020 |
| C        | 60  | 2020 |
| A        | 70  | 2020 |
(2155 rows affected)

Total 8 unique categories.
Total years is 3.

---

- We will group data by year and category and calculate the sum of quantity.

```tsql
SELECT Year, Category, SUM(Qty) AS TotalQty 
FROM Sales GROUP BY Year, Category;
```
(24 rows affected)

---

- We will use aggregate function with over to do same thing.

```tsql
SELECT DISTINCT Year, Category, SUM(Qty) OVER(PARTITION BY Year, Category) AS TotalQty
FROM Sales;
```
(24 rows affected)

---

- We will use `Pivot` to rotate the table-value.
- We have 3 seperate 

```tsql
SELECT CATEGORY, [2019], [2020], [2021]
FROM
(
    SELECT Category, Qty, Year
    FROM Sales
) AS SourceTable -- Grouping
PIVOT
(
    SUM(Qty) -- Aggregating
    FOR Year IN ([2019], [2020], [2021]) -- Spreading
) AS PivotTable;
```
(8 rows affected)


- Let's assume saved the result in a table `SalesPivot`.

| CATEGORY | 2019 | 2020 | 2021 |
|----------|------|------|------|
| A        | 40   | 70   | NULL |
| B        | 20   | 50   | NULL |
| C        | 40   | 60   | 78   |


- Unpivot SalesPivot table.
- Null values are not included in the result set.

```tsql
SELECT CATEGORY, YEAR, QTY -- YEAR and QTY are coming from UnpivotTable
FROM SalesPivot 
UNPIVOT
(
    QTY FOR YEAR IN ([2019], [2020], [2021])
) AS UnpivotTable;
```
(24 rows affected)

| CATEGORY | YEAR | QTY |
|----------|------|-----|
| A        | 2019 | 40  |
| A        | 2020 | 70  |
| B        | 2019 | 20  |
| B        | 2020 | 50  |
| C        | 2019 | 40  |
| C        | 2020 | 60  |
| C        | 2021 | 78  |


# Grouping Sets
- Grouping sets are used to group the data by multiple columns.
- if you need to group by different attributes at the same time, for example to report at different levels, you would normally need multiple queries combined with UNION ALL.
- if you need to produce aggregates of multiple groupings in the same query, you can use the GROUPING SETS subclause of the GROUP BY clause.
- Grouping sets are an alternative to using UNION ALL to combine multiple GROUP BY clauses.

Syntax:
```tsql
SELECT <column list with aggregate(s)>
FROM <source>
GROUP BY 
GROUPING SETS(
    (<column_name>), --one or more columns
    (<column_name>), --one or more columns
    () -- empty parentheses if aggregating all rows
);
```

---

```tsql
SELECT Category, Cust, SUM(Qty) AS TotalQty
FROM Sales.CategorySales
GROUP BY 
    GROUPING SETS((Category),(Cust),())
ORDER BY Category, Cust;
```

- First set grouped te result by Category.

| Category | TotalQty |
|----------|----------|
| A        | 100      |
| B        | 70       |
| C        | 100      |

- Second set grouped the result by Cust.

| Cust | TotalQty |
|------|----------|
| C1   | 50       |
| C2   | 70       |
| C3   | 100      |
| C4   | 50       |

- Third set grouped the result by all rows.

| TotalQty |
|----------|
| 270      |


- Grouping sets will union all the results.

| Category | Cust | TotalQty |
|----------|------|----------|
| NULL     | NULL | 270      |
| A        | NULL | 100      |
| B        | NULL | 70       |
| C        | NULL | 100      |
| NULL     | C1   | 50       |
| NULL     | C2   | 70       |
| NULL     | C3   | 100      |
| NULL     | C4   | 50       |


# Rollup And Cube
- Rollup and Cube are used to group the data by multiple columns.
- Therefore, CUBE and ROLLUP can be thought of as shortcuts to GROUPING SETS.

---

- CUBE will determine all possible combinations and output groupings.

```tsql
SELECT Category, Cust, SUM(Qty) AS TotalQty
FROM Sales.CategorySales
GROUP BY CUBE(Category,Cust);
```

| Category | Cust | TotalQty |
|----------|------|----------|
| NULL     | NULL | 270      |
| A        | C1   | 10       |
| A        | C2   | 20       |
| A        | C3   | 30       |
| A        | C4   | 40       |
| B        | C1   | 50       |
| B        | C2   | 20       |
| A        | NULL | 100      |
| B        | NULL | 70       |
| C        | NULL | 100      |
| NULL     | C1   | 50       |
| NULL     | C2   | 70       |
| NULL     | C3   | 100      |
| NULL     | C4   | 50       |


---

- ROLLUP will determine all possible combinations and output groupings.

```tsql
SELECT Category, Subcategory, Product, SUM(Qty) AS TotalQty
FROM Sales.ProductSales
GROUP BY ROLLUP(Category,Subcategory, Product);
```

This query will return combinations:
- (Category, Subcategory, Product)
- (Category, Subcategory)
- (Category)
- ()

> The order in which columns are supplied matters. The ROLLUP function will generate subtotals for the columns in the order they are supplied.


