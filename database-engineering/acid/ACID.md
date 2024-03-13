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
- Consistency is the property of a transaction that guarantees that the database remains in a consistent state before and after the transaction.

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
    - Read Committed
      - This is the default isolation level in SQL Server. Each query in a transaction only sees committed changes by other transactions.
      - It allows non-repeatable reads and phantom reads.
    - Repeatable Read
      - This isolation level prevents dirty reads and non-repeatable reads and lost updates.
      - When we read data, it locks the rows that we read. So, other transactions can't modify the data that we read.
    - Snapshot
      - This isolation level uses row versioning to provide transaction-level read consistency.
      - It doesn't lock the table or row when reading data. It reads the data from the version store.
      - If we have committed transactions, we won't see the changes until we commit our transaction.
    - Serializable
      - This is the highest isolation level. It locks the table when we read data.
      - Other transactions can't modify or insert that table until we commit our transaction.
      - It prevents dirty reads, non-repeatable reads, phantom reads, and lost updates.

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