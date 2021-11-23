Transaction Isolation levels

Problems:
Dirty Read Problem
Non-Repeatable Reads Problem
Lost Updates Problem
Phantom Reads Problem

Transaction Isolation levels:
Read uncommitted
Read committed (solves dirty-read problem)
Repeatable read (solves dirty-read, non-repeatable reads and lost-updates problem)
Serializable (also solves phantom-reads problem)

Read operations use Shared locks
Write/Update operations use Exclusive locks

Dirty read problem:
* Regardless of the transaction isolation level, a transaction always acquires an exclusive lock on the data it modifies and holds the lock until the end of the transaction.
* shared locks are not acquired at all at the read uncommitted isolation level

w1[x] .. r2[x] .. (a1, c2 any order)

Here transaction 2 does not ask for a shared lock, in case of read uncommitted level. Hence it can read uncommitted data.

Non repeatable reads problem:
* shared locks are released as soon as SELECT query execution completes

r1[P] .. w2[P] (c2 anywhere from here) r1[P] c1

Same reads of first transaction are different. This happens with both read uncommitted and read committed isolation levels.

Lost updates problem:

r1[x] .. r2[x] .. w2[x+2000] c2 w1[x+1000] c1

Here 2000 of 2nd transaction is lost. Root cause is the shared lock is released as soon as the SELECT query completes. This happens with both read uncommitted and read committed isolation levels.

* Transaction holds a shared lock until end of transaction in Repeatable read isolation level. So repeatable read isolation level solves non-repeatable reads problem.
* Repeatable read isolation level only PREVENTS lost updates problem but there will be a deadlock issue as shared lock is held until end of transaction. S1[x] .. S2[x] .. r1[x] .. r2[x] .. (both transactions are waiting for the other to release the shared lock, hence deadlock. One of them will be terminated. This is fine as order of api execution doesn’t matter.)
* Terminated transaction will throw error `error: Transaction was deadlocked on lock resources with another process and has been chosen as the deadlock victim. Rerun the transaction.` The backend logic must have a retry mechanism for transactions that were aborted due to deadlocks.

Phantom reads problem:-
Phantom read means that several identical SELECT queries executed within the same transaction return a different number of rows. How’s it different from non-repeatable read problem? Non-repeatable read problem only modifies the value of a row. Phantom read inserts/deletes a row.

S1[P] .. R1[P] .. E2[new] .. W2[new] .. c2 .. R1[P] (this time row count is different as new entry satisfies P)

But how is write possible with shared lock? Shared lock is only on the rows satisfying the predicate P. Exclusive lock is only on the row being inserted or deleted. Also row can’t be deleted in repeatable read isolation level as shared lock is not released until transaction completes. Row can only be deleted in read committed isolation level.
[https://social.msdn.microsoft.com/Forums/sqlserver/en-US/699ec3f8-e375-43b8-913c-0fcd3679cd2f/does-quotphantom-readsquot-issue-only-occur-with-insert-operation?forum=sqldatabaseengine]

* Serializable is the highest isolation level and prevents all possible types of concurrency phenomena, but the serializable level decreases performance and increases the likelihood of deadlocks.
* The serializable isolation level forces queries with ranged predicates to acquire range locks to avoid phantom reads.
* There are shared and exclusive range locks. RS1[P] .. R1[P] .. RE2[new] (waits) .. R1[P] (same) .. c1 .. W2[new] .. c2 
      RS - Range Shared lock, RE - Range Exclusive lock




FAQs:
* Is every SQL query a transaction?:- All individual SQL Statements, (with rare exceptions like Bulk Inserts with No Log, or Truncate Table) are automatically "In a Transaction" whether you explicitly say so or not.. (even if they insert, update, or delete millions of rows). 
* What is default isolation level of sql db?:- READ COMMITTED is the default isolation level for SQL Server. It prevents dirty reads by specifying that statements cannot read data values that have been modified but not yet committed by other transactions. 
* When are write changes in transaction reflected in db?:- Changes are not reflected in db until commit is executed. So queries of other transaction can’t see the updates of a transaction until it is committed. 
* In repeatable read isolation level, when exactly is the shared lock acquired?



