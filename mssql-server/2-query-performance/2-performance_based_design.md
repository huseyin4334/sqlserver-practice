# Normalization
- Database normalization is a design process used to organize a given set of data into tables and columns in a database. 
- Each table should contain data relating to a specific ‘thing’ and only have data that supports that same ‘thing’ included in the table. 
- The goal of this process is to reduce duplicate data contained within your database, to reduce performance degradation of database inserts and updates. 
- For example, a customer address change is much easier to implement if the only place the customer address is stored is in the Customers table. 
- The most common forms of normalization are first, second, and third normal form.

> Details: https://www.freecodecamp.org/news/database-normalization-1nf-2nf-3nf-table-examples/

> https://learn.microsoft.com/en-us/office/troubleshoot/access/database-normalization-description


## First Normal Form (1NF)
- Create a separate table for each set of related data.
- Eliminate repeating groups in individual tables.
- Create a primary key for each table.

You shouldn't use multiple columns in a single table to store similar data.
If we have same values in multiple rows, we should create a new table and move the redundant data to that table. We should connect the two tables with a primary key.


### Example

| EmployeeID | EmployeeName | JobCode | Job    | StateCode | HomeState |
|------------|--------------|---------|--------|-----------|-----------|
| 1          | John Doe     | J02     | Chef   | 26        | New York  |
| 2          | Jane Doe     | J01     | Waiter | 26        | New York  |
| 3          | Jim Doe      | J02     | Chef   | 56        | Michigan  |

- This table is in atomic form. Because each column uniquely identifies a row.


## Second Normal Form (2NF)
- The table must be in 1NF. 
- All non-key attributes should be fully dependent on the entire primary key.
- Composite key: A key that consists of more than one column.
  - If a table has a composite key, all columns in the table must be dependent on the entire key, not just part of it.
  - For example, if a table has a composite key of (A, B), then all columns in the table must be dependent on both A and B, not just A or B.

### Example

| EmployeeId | JobCode |
|------------|---------|
| 1          | J02     |
| 2          | J01     |
| 3          | J02     |

| EmployeeId | Name     | StateCode | HomeState |
|------------|----------|-----------|-----------|
| 1          | John Doe | 26        | New York  |
| 2          | Jane Doe | 26        | New York  |
| 3          | Jim Doe  | 56        | Michigan  |

| JobCode | Job    |
|---------|--------|
| J02     | Chef   |
| J01     | Waiter |

- In this example, the `Employee` table has a composite key of `EmployeeId` and `JobCode`.
- The `Employee` table is in 2NF because all columns are dependent on the entire key.


## Third Normal Form (3NF)
- The table must be in 2NF. 
- There should be no transitive dependencies between non-key attributes.
- The difference between 2NF and 3NF is that in 3NF, non-key attributes should not depend on other non-key attributes.

### Example

| EmployeeId | JobCode |
|------------|---------|
| 1          | J02     |
| 2          | J01     |
| 3          | J02     |

| EmployeeId | Name     | StateCode |
|------------|----------|-----------|
| 1          | John Doe | 26        |
| 2          | Jane Doe | 26        |
| 3          | Jim Doe  | 56        |

| JobCode | Job    |
|---------|--------|
| J02     | Chef   |
| J01     | Waiter |

| StateCode | HomeState |
|-----------|-----------|
| 26        | New York  |
| 56        | Michigan  |

- In this example, HomeState is dependent on StateCode but not dependent on EmployeeId.
- Because of that, the `Employee` table is not in 3NF.
- To make it 3NF, we should create a new table for `StateCode` and `HomeState`.


## Denormalization
While the third normal form is theoretically desirable, it isn't always possible for all data. 
In addition, a normalized database doesn't always give you the best performance. 
Normalized data frequently requires multiple join operations to get all the necessary data returned in a single query. 
There's a tradeoff between normalizing data when the number of joins required to return query results has high CPU utilization, and denormalized data that has fewer joins and less CPU required, but opens up the possibility of update anomalies.


> Todo: Star and snowflake schema














