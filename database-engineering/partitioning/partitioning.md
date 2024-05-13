# Partitioning
- Partitioning is a database design technique that improves performance, manageability, and availability of a database by distributing data among multiple tables.
- Partitioning is used to divide large tables into smaller, more manageable pieces called partitions.
- Each partition is a separate table that contains a subset of the data from the original table.


## Partitioning Types
- Partitioning can be done based on a range, list, hash, or composite of these methods. So, Partitioning is done based on rules.
  - Horizontal Partitioning: Rows are divided into partitions.
    - Range Partitioning: Rows are divided into partitions based on a range of values.
    - List Partitioning: Rows are divided into partitions based on a list of values.
    - Hash Partitioning: Rows are divided into partitions based on a hash function.
    - Composite Partitioning: Rows are divided into partitions based on a combination of range, list, and hash partitioning.
  - Vertical Partitioning: Columns are divided into partitions.

## Partitioning vs Sharding
- Partitioning is distributing the data into multiple tables based on a rule. Sharding is distributing the data into multiple servers based on a rule.
- Your database names change in partitioning. But your database names don't change in sharding.

## Example
- We have 100 million rows in the table. So we decided to partition the table based on the range of the g column.
- g column has different 100 values.
- We decided to create 5 partitions. Let coding;

``` schell
docker run --name postgres -e POSTGRES_PASSWORD=postgres -p 5432:5432 -d postgres
# create a postgres container.

docker exec -it postgres bash
# connect to the postgres container bash
```

### Normal Scenario

```sql

create table grades(
    id serial primary key,
    g integer not null
);
-- create a table

insert into grades(g) select floor(random() * 100) from generate_series(1, 10000000);
-- insert 1000000 rows into the table

create index grades_g_idx on grades(g);

explain analyze select * from grades where g = 50;
-- Bitmap Heap Scan on grades ...
    -- Bitmap Index Scan on grades_g_idx ...
    -- Planning Time: 0.073 ms
    -- Execution Time: 2031.251 ms

explain analyze select * from grades where g between 50 and 60;
-- Finalize Aggregate ... -> Finalize aggregate means aggregating 2 or more stages.
    -- Gather ... -> Gather means leader node for workers.
        -- Partial Aggregate ...  -> Partial aggregate means 1 calculator stage.
            -- Bitmap Index Only Scan on grades_g_idx ... -> Index Only Scan means that the database used only the index to get the result.
    -- Planning Time: 0.073 ms
    -- Execution Time: 2031.251 ms
    -- https://access.crunchydata.com/documentation/postgresql96/9.6.24/parallel-plans.html
```

### Partitioning Scenario

```sql

create table grades_parts(
    id serial primary key,
    g integer not null
) partition by range(g);
-- partition the table based on the range of the g column.

-- We create 5 partitions. like ... including indexes means that we will copy the indexes of the grades_parts table.
create table g035 (like grades_parts including indexes);
create table g3560 (like grades_parts including indexes);
create table g6080 (like grades_parts including indexes);
create table g80100 (like grades_parts including indexes);

alter table grades_parts attach partition g035 for values from (0) to (35);
alter table grades_parts attach partition g3560 for values from (35) to (60);
alter table grades_parts attach partition g6080 for values from (60) to (80);
alter table grades_parts attach partition g80100 for values from (80) to (100);
-- we attach the partitions to the grades_parts table.
-- Database will decide which partition will be used based on the g column.

create index grades_parts_g_idx on grades_parts(g);
-- when we create an index on the grades_parts table, the index will be created on the partitions with partition table specific.
-- Every table will have its own index.
-- g035_g_idx, g3560_g_idx, g6080_g_idx, g80100_g_idx

insert into grades_parts select * from grades;
-- insert 1000000 rows into the table
-- Database will decide which value will be inserted into which partition based on the g column.

explain analyze select * from grades_parts where g = 50;
-- Bitmap Heap Scan on g3560 ...
    -- Bitmap Index Scan on g3560_g_idx ...
    -- Planning Time: 0.709 ms
    -- Execution Time: 1788.251 ms

select  pg_relation_size(oid), relname from pg_class order by pg_relation_size(oid) desc;
/*
    pg_relation_size | relname
    ----------------- | -------
    362479616         | grades
    1266836736        | g035
    90652672          | g3560
    ....              | g6080
    ....              | g80100
    69361664          | grades_g_idx
    24289280          | g035_g_idx
    17367040          | g3560_g_idx
    ....              | g6080_g_idx
    ....              | g80100_g_idx
 */
 
 -- If we have some limitations on the resources, we can use partitioning to reduce the size of the tables.


show ENABLE_PARTITION_PRUNING;
-- It's true by default. It means that the database will use the partitioning to get the result.
-- It will trust to the partitioning rules. And search the result in which partition table based on the partitioning rules.
-- If we set it to false, the database will search all the partition tables to get the result.

set ENABLE_PARTITION_PRUNING = off;
```

## Pros and Cons of Partitioning

### Pros
- Partitioning improves performance by reducing the size of the tables.
- Scans only the necessary partitions to get the result.
- Easy bulk loading and deleting data.
- Easy to manage the data.
- Archive old data that are barely accessed into cheap storage.

### Cons
- Updated rows can be moved to another partition. It can cause performance issues or fails.
- Inefficient queries can be run on the database. It can cause scan all the partitions.
- Schema changes can be hard to manage.
