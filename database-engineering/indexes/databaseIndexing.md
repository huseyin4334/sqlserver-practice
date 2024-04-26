# INDEXING
- Indexing is a way to optimize the performance of a database by minimizing the number of disk accesses required when a query is processed.

## Populate Multiple Rows
### Create a container
```bash
# create a postgres container. 
# detach from the terminal and run in the background
# expose the port 5432 on the host machine
# name the container postgres-db
docker run -e POSTGRES_PASSWORD=secret -d -p 5432:5432 --name postgres-db postgres
# connect to the container for executing commands
docker exec -it postgres-db psql -U postgres
```
### Create Table And Insert Data
```sql
create table temp(
    id serial primary key,
    name varchar(100),
);

insert into temp values select random() * 100,  "ZF" + random() * 100 from generate_series(1, 1000000); 
```

## Indexing
Databases have different types of indexes. The most common types are:
- B-Tree
- Hash
- GiST
- ...

You can read more about the types of indexes in the documentation of postgresql. (Beginner)
- https://vladmihalcea.com/postgresql-index-types/

You can read default indexes in the documentation of the database you are using.
- https://vladmihalcea.com/default-database-key-indexing/

We will discuss how is working indexing in general.
```sql

/* 
    primary key is a default index in the database.
    We will have 1000000 rows in the table.
*/
CREATE TABLE temp(
    id serial primary key,
    name varchar(100),
);

explain analyze select id from temp where id = 1000000;
-- Index Only Scan using temp_pkey on temp ...
-- EXECUTION TIME: 0.657 ms

explain analyze select name from temp where id = 5000;
-- Index Scan using temp_pkey on temp ... (NOT ONLY)
-- EXECUTION TIME: 2.548 ms
-- Because Database first scanned the index and later it's gone to page to get the name.

explain analyze select name from temp where id = 5000;
-- EXECUTION TIME: 0.157 ms
-- Database used cache to get the name.

explain analyze select id from temp where name = 'ZF';
-- Gather Seq Scan ... -> Parallel Seq Scan on temp ... 
-- EXECUTION TIME: 3199.734 ms
-- Database scanned all pages to get the id. And we don't have an index on the name.

create index temp_name_idx on temp(name);

explain analyze select id from temp where name = 'ZF';
-- Bitmap Heap Scan on temp ... -> Bitmap Index Scan on temp_name_idx ...
-- EXECUTION TIME: 47.089 ms
-- Bitmap heap scan is a way to optimize the performance of the database.

explain analyze select name from temp where name like '%ZF%';
-- Gather Seq Scan ... -> Parallel Seq Scan on temp ...
-- EXECUTION TIME: 1675.734 ms
-- Database didn't use the index because we have a wildcard at the beginning of the string.
-- Database don't use indexes when we have a wildcard at the beginning of the string. Because indexes working left to right.
-- https://use-the-index-luke.com/sql/where-clause/searching-for-ranges/like-performance-tuning#:~:text=The%20LIKE%20operator%20works%20on,prevent%20using%20indexes%20for%20LIKE%20.
-- Databases offers full-text search for this kind of queries.
```

### Scan Types
- Index Only Scan
  - This scan is used just index pages. It's faster than the index scan. Because it doesn't need to go to the table page.
- Index Scan
  - This scan is used index pages and table pages. It's using when filtering with the index and selecting columns from the table.
- Bitmap Heap Scan
  - This scan is used when we have multiple indexes. It's used to combine the indexes. It's used to optimize the performance of the database.
- Bitmap Index Scan
  - This scan is used to get the index pages and combine them with the bitmap heap scan.
- Seq Scan
  - This scan is used when we don't have an index on the table. It's used to scan all pages of the table.
- Parallel Seq Scan
    - This scan is used to scan all pages of the table in parallel. It's used to optimize the performance of the database.

## Sql Query Planner And Optimizer
- The SQL query planner is a component of a database system that analyzes a query and determines the most efficient way to execute the query.
- The SQL query optimizer is a component of a database system that takes the query plan generated by the query planner and further optimizes it to improve performance.
- The query planner and optimizer use statistics about the data in the database to make decisions about how to execute a query.
- How the query planner make a decision about scan type?
  - The query planner uses the cost-based optimization technique to estimate the cost of different query execution plans and choose the one with the lowest cost.
  - It uses the statistics about the data in the database to calculate the cost of different scan types.

```sql

create table temp(
    id serial primary key,
    name varchar(100),
    g integer not null
);

-- insert data

create index temp_g_idx on temp(g);

-- we have 50000000 rows in the table
-- we have primary key index and g index

explain select * from temp;
-- Seq Scan on temp (cost=0.00..1040000.00 rows=50000000 width=12)
-- cost have two values. The first value is how much time took to get the first page. The second value is how much time took total time.
-- It's calculated by the query planner.
-- rows is the number of rows returned by the query.
-- width is the size of the returned 1 row in bytes.


explain select * from temp order by g;
-- Index Scan using temp_g_idx on temp (cost=0.43..1040000.00 rows=50000000 width=12)
-- The query planner got rows. But it's not sorted. It's used the index to sort the rows.

execute select * from temp order by name;
-- Gather Merge on temp (cost=102345.74..222345.74 rows=50000000 width=12)
-- Workers Planned: 2
-- -> Sort (cost=102345.72..112345.72 rows=50000000 width=12)
--      Sort Key: name
--      -> Parallel Seq Scan on temp (cost=0.00..2180201.00 rows=50000000 width=12)

-- The query planner used the parallel seq scan to get the rows. And later it's sorted the rows by name.

explain select id from temp;
-- Seq Scan on temp (cost=0.00..1040000.00 rows=50000000 width=4)
-- The query planner used the seq scan to get the id. But we have index on the id. It's not used the index.
```

## Key And Non-Key Indexes
- Key indexes are created by a column indexes.
- Non-key indexes are included columns in the index.

```sql

create table temp(
    id serial primary key,
    name varchar(100),
    g integer not null
);

create index temp_g_idx on temp(g) include(name);

explain (analyze, buffers) select name from temp where g = 100;
-- Index Only Scan using temp_g_idx on temp (cost=0.43..8.45 rows=1 width=100) (actual time=0.013..0.013 rows=1 loops=1)
-- Index Cond: (g = 100)
-- Heap Fetches: 0 **(If we have heap fetches, it's gone to the table page to get the name)**
-- Buffers: shared hit=3 read=0 (If read count bigger than 0, It come from cache)
-- Execution Time: 0.025 ms
-- The query planner used the index to get the name. And it's used the index only scan. It's not gone to the table page.
```


## Combining Indexes

```sql

create table temp(
    a integer not null,
    b integer not null,
    c integer not null
);

create index temp_a_idx on temp(a);
create index temp_b_idx on temp(b);


explain analyze select c from temp where a = 100;
-- Used index temp_a_idx to get the rows. It's gone to the table page to get the c column.

explain analyze select c from temp where a = 100 and b = 200;
-- Used bitmap heap scan on temp_a_idx, temp_b_idx and bitmapAnd.

explain analyze select c from temp where a = 100 or b = 200;
-- Used bitmap heap scan on temp_a_idx, temp_b_idx and bitmapOr.

drop index temp_a_idx;
drop index temp_b_idx;

create index temp_ab_idx on temp(a, b);

explain analyze select c from temp where a = 100;
-- Used index temp_ab_idx to get the rows. It's gone to the table page to get the c column.

explain analyze select c from temp where b = 200;
-- Used Seq Scan on temp. Because index not usable for this query. (Indexes working left to right.)

explain analyze select c from temp where a = 100 and b = 200;
-- Used index temp_ab_idx to get the rows. It's gone to the table page to get the c column.

explain analyze select c from temp where a = 100 or b = 200;
-- Used Seq Scan on temp. Because index not usable for this query. (Indexes working left to right.)

create index temp_b_idx on temp(b);

explain analyze select c from temp where a = 100 or b = 200;
-- Used bitmap heap scan on temp_ab_idx, temp_b_idx and bitmapOr.

expalin analyze select c from temp where b = 200;
-- Used index temp_b_idx to get the rows. It's gone to the table page to get the c column.
```

> We combined a and be in 1 index. So query used jost 1 index. But before we had 2 indexes. So Planner used 3 function to get the rows.


## Concurrent Index Creation
- The database allows creating indexes concurrently. It's used to create indexes without blocking the table.

```sql
create index concurrently temp_name_idx on temp(name);
```

## Bloom Filters
- Bloom filters are a probabilistic data structure that is used to test whether an element is a member of a set.
- Bloom filters are used in databases to reduce the number of disk accesses required when a query is processed.
- Bloom filters are used to check if a row exists in a table without having to read the entire table.
- It's working like a hash index. I will explain with a username search example.
  - We have 64 bit array.
  - When we write "paul" to the table. We will hash the "paul" and get the hash value. 
    - Later hash("paul") % 64 will give us the index of the array. We will set the index to 1.
  - When we read "paul" from the table. We will hash the "paul" and get the hash value. 
    - Later hash("paul") % 64 will give us the index of the array. We will check the index. If it's 1, we will return true. If it's 0, we will return false.