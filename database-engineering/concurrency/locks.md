# Locks
- Locks are used to prevent data corruption and ensure data integrity in a multi-user environment.
- Locks are used to prevent two or more users from updating, reading or deleting the same data at the same time.

## Types of Locks
- **Shared Locks (S Locks)**: Multiple transactions can hold a shared lock on the same resource.
- **Exclusive Locks (X Locks)**: Only one transaction can hold an exclusive lock on a resource.

### Differences between Shared and Exclusive Locks
- **Shared Locks**: Used for read operations. Multiple transactions can hold a shared lock on the same resource.
- **Exclusive Locks**: Used for write operations. Only one transaction can hold an exclusive lock on a resource.

- So if I read something, I use shared lock. Because No one can not write while I read. But Other shared locks can read while I read too.
- If I write something, I use exclusive lock. Because No one can not read or write while I write.

## Deadlocks
- Deadlocks occur when two or more transactions are waiting for each other to release locks.
- Deadlocks can be prevented by using a timeout or by killing one of the transactions.

### Example

```sql
create table employees (
    id int primary key,
    name varchar(100),
    salary int
);


-- Transaction 1
BEGIN TRANSACTION;
insert into employees (id, name, salary) values (1, 'John', 1000); -- First insert
-- We will see insert 1
insert into employees (id, name, salary) values (2, 'Jane', 3000); --Third insert
-- We will stuck here
-- When finish fourth insert, This insert will be done. Because 4. insert will be fail and transaction will be rollback.


-- Transaction 2
BEGIN TRANSACTION;
insert into employees (id, name, salary) values (2, 'Jane', 3000); -- Second insert
-- We will see insert 1

insert into employees (id, name, salary) values (1, 'John', 1000); -- Fourth insert
-- We will get a deadlock error in here
-- Error: Deadlock detected
-- Detail: Process 56 waits for ShareLock on transaction 1; blocked by process 55.
-- Hint: See server log for query details.
-- Context: while inserting index tuple (0,2) in relation "employees"

```

- In this example, Transaction 1 and Transaction 2 are waiting for each other to release locks. So, we get a deadlock error.

```sql
create table employees (
    id int primary key,
    name varchar(100),
    salary int
);


-- Transaction 1
BEGIN TRANSACTION;
insert into employees (id, name, salary) values (1, 'John', 1000); -- First insert
-- We will see insert 1
insert into employees (id, name, salary) values (2, 'Jane', 3000); --Third insert
-- We will stuck here
-- When finish fourth command, This insert will be done. Because 4.command will get back all changes.


-- Transaction 2
BEGIN TRANSACTION;
insert into employees (id, name, salary) values (2, 'Jane', 3000); -- Second insert
-- We will see insert 1
rollback; -- Fourth command
```

```sql
create table employees (
    id int primary key,
    name varchar(100),
    salary int
);


-- Transaction 1
BEGIN TRANSACTION;
insert into employees (id, name, salary) values (1, 'John', 1000); -- First insert
-- We will see insert 1
insert into employees (id, name, salary) values (2, 'Jane', 3000); --Third insert
-- We will stuck here
-- When finish fourth command, This insert will get an error. Because 4.command will commit all changes.
-- Error: duplicate key value violates unique constraint "employees_pkey"
-- Detail: Key (id)=(1) already exists.


-- Transaction 2
BEGIN TRANSACTION;
insert into employees (id, name, salary) values (2, 'Jane', 3000); -- Second insert
-- We will see insert 1
commit; -- Fourth command
```

#### Two Phase Locking
- Two-phase locking is a concurrency control method that guarantees serializability.
- In two-phase locking, a transaction can acquire locks but cannot release any lock until it has acquired all the locks it needs.
- So, Two-phase locking is a method to prevent deadlocks. It's use exclusive lock for write operations and shared lock for read operations.

##### Problem

```sql

-- Transaction 1
BEGIN TRANSACTION;

select * from employees where id = 1; -- We will see 1 John 1000

update employees set salary = 2000 where id = 1; -- First update
-- It's ok
                                                 
select * from employees where id = 1; -- We will see 1 John 2000. We say, yes everything is ok.                                         

commit; -- first commit

-- It's called after the second commit. We changed value in the same time. I control value and i see the value is changed.
select * from employees where id = 1; -- We will see 1 John 3000. We say, yes everything is ok.


-- Transaction 2
BEGIN TRANSACTION;

select * from employees where id = 1; -- We will see 1 John 1000

update employees set salary = 3000 where id = 1; -- Second update
-- It's waiting for the first transaction to release the lock.
-- This will be done after the first transaction is committed.
                                                 
select * from employees where id = 1; -- We will see 1 John 3000. We say, yes everything is ok.

commit; -- second commit
```

##### Solution
```sql

-- Transaction 1
BEGIN TRANSACTION;

select * from employees where id = 1 for update; -- We will see 1 John 1000. But I locked the rows with exclusive lock. So, no one can not read or write this row.

update employees set salary = 2000 where id = 1; -- First update
-- It's ok
                                                 
select * from employees where id = 1; -- We will see 1 John 2000. We say, yes everything is ok.                                         

commit; -- first commit


-- Transaction 2
BEGIN TRANSACTION;

select * from employees where id = 1; -- We will wait in here. Because transaction 1 locked the row with exclusive lock. So, we can not read or write this row.
-- But this select will work after the first transaction is committed.
-- We will see 1 John 2000. We will say ok value is changed by someone. Don't change or change.

update employees set salary = 3000 where id = 1; -- Second update
-- It's waiting for the first transaction to release the lock.
-- This will be done after the first transaction is committed.
                                                 
select * from employees where id = 1; -- We will see 1 John 3000. We say, yes everything is ok.

commit; -- second commit
```