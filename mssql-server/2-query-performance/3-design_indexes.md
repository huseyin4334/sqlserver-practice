# Index
- Indexes are used to speed up the query performance.

- Clustered Index
  - Clustered Indexes physically sort the data rows in the table based on the index key.
  - A table can have only one clustered index.
  - Clustered index is like a pointer to the data rows.
  - Clustered index is automatically created when we create a primary key.
- Non-Clustered Index
  - Non-Clustered Indexes are stored separately from the data rows.
  - A table can have multiple non-clustered indexes.
  - Non-clustered index is like a pointer to the primary key.
  - When we need to extra columns, index will be key lookup with the clustered index via the primary key.

## Examples
- Predicates in Index seek
  - Predicates are used to filter the rows.
  - Seek predicates are used to filter the rows in the index with the given values.

```sql
-- We have an index IDX_STORE_CODE_AND_DEALER_CODE
select * from DEALER d where d.DEALER_CODE like '%456%' and MERCHANT_CODE= 'USA'

-- This query used the index IDX_STORE_CODE_AND_DEALER_CODE
-- The query optimizer used the index to filter the rows that have '456' in the DEALER_CODE column. 
-- Because data statistics show that the index is selective for the '456' value.

-- Index seek operation
    -- MERCHANT_CODE = 'USA' (Seek predicate)
-- After seek operation, filtered data with %456% in DEALER_CODE column. (predicate)
-- Key lookup operation
    -- More value is needed from the table.
    -- Query optimizer used the clustered index to get the extra columns with found primary key from the IDX_STORE_CODE_AND_DEALER_CODE index.
-- Nested loop join
    -- Join the filtered data with the extra columns from the table.
```



```tsql
CREATE NONCLUSTERED INDEX IDX_STORE_CODE_AND_DEALER_CODE ON DEALER (MERCHANT_CODE, DEALER_CODE)
INCLUDE (DEALER_NAME, ADDRESS, CITY, STATE, ZIP_CODE)

SELECT DEALER_NAME, ADDRESS, CITY, STATE, ZIP_CODE FROM DEALER WHERE MERCHANT_CODE = 'USA' AND DEALER_CODE LIKE '%456%'

-- Index seek operation
    -- MERCHANT_CODE = 'USA' (Seek predicate)
-- After seek operation, filtered data with %456% in DEALER_CODE column. (predicate)

-- We got the items with just the index. We don't need to go to the table to get the extra columns.
-- Because we included the extra columns in the index.
```