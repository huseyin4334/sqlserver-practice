# ACID

## Transaction
- A transaction is a sequence of operations performed as a single logical unit of work.
- A collection of queries that are executed as a single unit.
- We have to use transactions to do anything in a database.

### Transaction Lifespan
- A transaction begins with BEGIN TRANSACTION and ends with COMMIT or ROLLBACK.
- BEGIN TRANSACTION: starts a new transaction.
- COMMIT: ends a transaction successfully.
  - It saves the changes to the database permanently.
- ROLLBACK: ends a transaction unsuccessfully.
  - A transaction can be rolled back if an error occurs.
  - It undoes the changes made in the database.
  - If transaction is unexpectedly terminated, all changes are rolled back too.

- So, transaction is simulation of a single unit of work. If simulation is failed, it won't be saved all simulated changes.

### Nature of Transactions
- Usually, transactions are used to change or modify data in a database. (Read-Write transaction)
- However, a transaction can also be used to only read data from a database. (Read-only transaction)
- When a transaction is read-only, it is guaranteed that the data will not change during the transaction.

## ATOMICITY
- Atomicity is the property of a transaction that guarantees that either all the queries is executed or none of it is.
- Transaction is an atom. It means that all queries of transaction a single unit of work.

## CONSISTENCY
- Consistency is the property of a transaction that guarantees that the database stays in a consistent state before and after the transaction.
- Defining the rules that the database must follow to be in a consistent state.
- So, any change maintains data integrity or is cancelled completely.

## ISOLATION
- Isolation is the property of a transaction that guarantees that the execution of a transaction is isolated from the execution of other transactions.
- But if we have a transaction that is not committed, But Other transaction committed and changed the same data. We can choose how to handle this situation.
  - Read Phenomena: This is a situation where a transaction reads data that is being modified by another transaction. (These all are just situations, we will control them with isolation levels.)
    - Dirty Read: A transaction reads data that has been modified by another transaction but not yet committed.
    - Non-Repeatable Read: A transaction reads the same data twice and gets different results.
    - Phantom Read: A transaction reads a set of rows that satisfy a search condition and another transaction inserts new rows that satisfy the same search condition.
    - Lost Update: A transaction reads a row, another transaction reads the same row, and the first transaction updates the row. The second transaction updates the row without knowing that the first transaction has updated it.
  
  - Isolation levels are used to handle this situation.
  - There are 4 isolation levels in SQL Server.
    - Read Uncommitted
      - This is the lowest isolation level. It doesn't look for any locks by any other transactions.
      - It allows dirty reads, non-repeatable reads, and phantom reads.
      - BEGIN TRANSACTION WITH ISOLATION LEVEL READ UNCOMMITTED
    - Read Committed
      - This is the default isolation level in SQL Server. Each query in a transaction only sees committed changes by other transactions.
      - It's not take any snapshot. When we start a transaction, we will see the committed changes by other transactions.
      - And we will continue to see the committed changes by other transactions until we commit the transaction.
      - BEGIN TRANSACTION WITH ISOLATION LEVEL READ COMMITTED
    - Repeatable Read (Snapshot Isolation Level)
      - This isolation level prevents dirty reads and non-repeatable reads and lost updates.
      - It's snapshots the committed or uncommitted data in the beginning of the transaction.
      - When other transactions inserted data and committed, we won't see uncommitted data in postgreSQL, but we will see it in other databases.
      - You will see the same data with the beginning of the transaction until commit the transaction. Because of that, you can see the uncommitted changes by other transactions.
      - BEGIN TRANSACTION WITH ISOLATION LEVEL REPEATABLE READ
    - Serializable
      - This is the highest isolation level. It's taking snapshot of all committed data in the beginning.
      - When we start transaction with this level, we won't see any committed or uncommitted changes by other transactions.
      - This is isolated from all other transactions. 
      - BEGIN TRANSACTION WITH ISOLATION LEVEL SERIALIZABLE
> **Critical Note:** When I assume we started normal transaction and insert 1 row. But we didn't commit.
> we have opened another transaction with repeatable read isolation level. We will see the inserted row in this transaction. But I didn't commit yet.
> I insert a row from normal transaction again. But I won't see inserted row in repeatable read transaction. Because we said repeatable read isolation level will show us the same data with beginning of the transaction.
> But if we use serializable isolation level, we won't see the inserted 2 rows in the second transaction. Because it's not showing us any uncommitted changes.

![repeatable-read](/../sources/repeatable-read.png)
![serialization](/../sources/serializable.png)

> **Critical Note:** When you use repeatable read isolation level, Every transaction can change the same table.
> That means first transaction change data and commit. Second transaction can change something too and commit.
> It will be ok.
> But when you use serializable isolation level, just 1 transaction can change the same table. 
> Other transactions have to begin after the first transaction is committed.

> **Critical Note:** With transaction 1 as READ COMMITTED, you can update a row in transaction 2 after you selected it in transaction 1. 
> With transaction 1 as REPEATABLE READ, you cannot update a row in transaction 2 after you selected it in transaction 1.
> Then, we prevent lost updates with REPEATABLE READ isolation level with locked the row.
> But with transaction 1 as SERIALIZABLE, you cannot update a row in transaction 2 after you selected table in transaction 1.
> Then, we prevent lost updates with SERIALIZABLE isolation level with locked the table.
 

> **Note:** If we don't want to changeable data while we are changing data;
> SELECT * FROM employees WHERE id = 1 FOR UPDATE; -- we lock the selected rows with exclusive lock in read committed isolation level.
> SELECT * FROM employees WHERE id = 1; -- If we start the repeatable read isolation level, this select will work like "**for update**" command. Because this isolation level uses exclusive lock default when update, delete, etc. commands.

![isolations](/../sources/isolations.png)

### Locks
- Locks are used to control the access of data by multiple transactions.
- Pessimistic Locking: 
  - Row level locks, table level locks, page level locks to avoid lost updates.
- Optimistic Locking:
  - It doesn't use locks to control the access of data.
  - It uses versioning to control the access of data.
  - It is used in the Snapshot isolation level.
- Repeatable Read isolation level uses row level locks to avoid lost updates.
- Serializable isolation level uses table level locks to avoid lost updates.

## DURABILITY
- Durability is the property of a transaction that guarantees that the changes made by a transaction are permanent and are not lost.
- For example, I have a transaction that inserts a row into a table. If the transaction is committed but database server is crashed, if i committed and i get COMMIT message, Database guarantee that the row is inserted into the table.
- Databases use log files to guarantee durability. 
  - Log files are used to keep track of all the changes made by the transactions.
  - If the database server crashes, the log files are used to recover the database.
- Since the crash happened during the commit the database cannot guarantee durability. The system is durable only when the commit is successful( the data is fully written to disk). That is why commit speeds are critical, the faster you can commit the lower the chances of such corruption.
- Durability techniques; (Default is Write-Ahead Logging (WAL) in SQL Server)
  - Write-Ahead Logging (WAL)
    - It is a technique that writes the changes to the transaction log before the changes are written to the database.
  - Shadow Paging (Asynchronous Snapshot)
    - It is a technique that creates a shadow copy of the database before the changes are made by the transaction. 
  - Journaling
    - It is a technique that writes the changes to the journal before the changes are written to the database. (Journal is a file that contains all the changes made by the transactions.) 
  - Log Structured File System (LSFS)
    - It is a file system that writes the changes to the transaction log before the changes are written to the database.
    - File system is a system that manages the storage of data in a computer.
  - ....

