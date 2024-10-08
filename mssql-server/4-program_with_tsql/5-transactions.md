# Transaction
- A transaction is a sequence of operations performed as a single logical unit of work. 
- A logical unit of work must exhibit four properties, called the ACID (Atomicity, Consistency, Isolation, and Durability) properties, to qualify as a transaction.
- Transactions encapsulate operations that must logically occur together, such as multiple entries into related tables that are part of a single operation.

- Transaction types:
  - Implicit Transaction
    - SQL Server automatically starts a new transaction after the current transaction is committed or rolled back.
  - Explicit Transaction
    - You can explicitly start a new transaction by using the BEGIN TRANSACTION statement.

> When we open an inner transaction, it is not a new transaction. It is a part of the outer transaction.
> If the outer transaction is rolled back, the inner transaction is also rolled back too.

---

Generally in a batch, the transaction is started implicitly line by line.
But when we handle transactions manually, we can start and end transactions explicitly.

> [Compare Batches And Transactions](https://learn.microsoft.com/en-us/training/modules/implement-transactions-transact-sql/3-compare-transactions-batches)

---

## XACT_ABORT
When SET XACT_ABORT is ON, if SQL Server raises an error, the entire transaction is rolled back. 
When SET XACT_ABORT is OFF, only the statement that raised the error is rolled back if the severity of the error is low.

```tsql
SET XACT_ABORT OFF;

BEGIN TRY
  BEGIN TRANSACTION;
    insert into table1 values (1, 'a');
    insert into table1 values (2, 'b');
    insert into table1 values (3, 'c'); -- Got error here
    insert into table1 values (4, 'd');
  COMMIT TRANSACTION;
END TRY
BEGIN CATCH
  PRINT 'Error occurred. Rolling back transaction.';
  ROLLBACK TRANSACTION;
END CATCH;

-- Select the table
select * from table1;
```

| id | name |
|----|------|
| 1  | a    |
| 2  | b    |


## XACT_STATE
`XACT_STATE()` returns the state of the current transaction.
Return values:
- 0: There is no active transaction.
- 1: There is an active transaction and the transaction has not been committed or rolled back.
- -1: The transaction is in an uncommittable state. The transaction has been rolled back.

```tsql
BEGIN TRY
  BEGIN TRANSACTION;
    insert into table1 values (1, 'a');
    insert into table1 values (2, 'b');
    insert into table1 values (3, 'c'); -- Got error here
    insert into table1 values (4, 'd');
  COMMIT TRANSACTION;
END TRY
BEGIN CATCH
  PRINT 'Error occurred. Rolling back transaction.';
  if XACT_STATE() <> 0
    BEGIN
      PRINT 'XACT_STATE: ' + CAST(XACT_STATE() AS VARCHAR(8));
      ROLLBACK TRANSACTION;
    END
END CATCH;

-- Select the table
select * from table1;
```

| id | name |
|----|------|

---

# Concurrency
- Concurrency is the ability of a database to allow multiple users to affect multiple transactions at the same time.

> Details: [Concurrency Control](https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide?view=sql-server-ver16)

## Optimistic Concurrency
At the start of the transaction, the initial state of the data is recorded.
Before the transaction is committed, the current state of table is compared with the initial state for changed rows. If the states are the same, the transaction is completed. 
If the states are different, the transaction rolled back.

By using optimistic locking, transactions don't block each other and the system runs more efficiently.

## Pessimistic Concurrency
Pessimistic concurrency control locks data as soon as it is read, and the data remains locked until the transaction is completed.
Any other transaction that tries to read or write to the data is blocked until the lock is released.

This can prevent large rollbacks, as seen in the previous example, but can cause queries to be blocked unnecessarily.

## Snapshot Isolation
READ_COMMITTED_SNAPSHOT_OFF is the default isolation level for SQL Server. READ_COMMITTED_SNAPSHOT_ON is the default isolation level for Azure SQL Database.

READ_COMMITTED_SNAPSHOT_OFF uses pessimistic concurrency control, while READ_COMMITTED_SNAPSHOT_ON uses optimistic concurrency control.
READ_COMMITTED_SNAPSHOT_OFF will hold locks on the effected rows until the transaction is completed (read or wrote rows).
When we read it lock with a shared lock -> Other transactions can continue to read the rows. But they can't write to the rows.
When we write it lock with an exclusive lock. -> Other transactions can read last committed version of the rows. But they can't write to the rows.

READ_COMMITTED_SNAPSHOT_ON takes a snapshot of the table.
We read and write in this snapshot.
When we commit the transaction, it compares the snapshot with the current state of table. If there is no change, it commits the transaction. 
If there is different from current state, it rolls back the transaction.

```tsql
ALTER DATABASE dbName SET READ_COMMITTED_SNAPSHOT ON;
```