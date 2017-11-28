+++
date = "2017-11-28T09:59:58+03:00"
draft = false
title = "PostgreSQL transaction isolation levels"
tags = ["database", "postgresql"]
+++

On previous post [about transaction isolation levels](/post/transaction_isolation_levels/) we considered what is 
transaction isolation level and how this influence the result we get from transaction. But from database to database 
details are diferent, so let's consider transaction isolation details for PostgreSQL.

Let's again repeat a little bit about transaction islolation level. 
The SQL standard defines four levels of transaction isolation. 
The most strict is `Serializable`, which is defined by the standard in a paragraph which says 
that any concurrent execution of a set of `Serializable` transactions is guaranteed to produce the same effect 
as running them one at a time in some order. 

The other three levels are defined in terms of phenomena, resulting from interaction between concurrent transactions, which must not occur at each level.

The phenomena which are prohibited at various levels are:

* **dirty read** - A transaction reads data written by a concurrent uncommitted transaction.
* **nonrepeatable read** - A transaction re-reads data it has previously read and finds that data has been modified by another transaction (that committed since the initial read).
* **phantom read** - A transaction re-executes a query returning a set of rows that satisfy a search condition and finds that the set of rows satisfying the condition has changed due to another recently-committed transaction.

## Only 3 isolation level internally

Internally, there are only three distinct isolation levels, which correspond to the levels `Read Committed`, 
`Repeatable Read`, and `Serializable`. 

When you select the level `Read Uncommitted` you really get `Read Committed`. 

`Phantom reads` are not possible in the PostgreSQL implementation of `Repeatable Read`, so the actual isolation level might be stricter than what you select. 
This is permitted by the SQL standard: the four isolation levels only define which phenomena must not happen, they do not define which phenomena must happen. 

The reason that PostgreSQL only provides three isolation levels is that this is the only sensible way to map the standard isolation levels 
to the MVCC architecture.

## Read commited (default)

A `SELECT` query (without a `FOR UPDATE/SHARE` clause) sees only data **committed before the query (not transaction!!) began**; 
it never sees either uncommitted data or changes committed during query execution by concurrent transactions. 
In effect, a SELECT query sees a snapshot of the database **as of the instant the query (not transaction!!) begins to run**.

However, `SELECT` does see the effects of previous updates executed within its own transaction, even though they are not yet committed. 
Also note that two successive `SELECT` commands **can see different data**, even though they are within a single transaction, 
if other transactions commit changes during execution of the first `SELECT`.

## Repeatable read

The `Repeatable Read` isolation level only sees data **committed before the transaction began**. This is the difference wit hprevious level - 
`Read commited` which sees data **before query began**. 
Thus, successive `SELECT` commands within a single transaction see the same data, i.e., 
they do not see changes made by other transactions that committed after their own transaction started.

Applications using this level must be prepared to retry transactions due to serialization failures. 
Note that only updating transactions might need to be retried; read-only transactions will never have serialization conflicts.

## Serializable Isolation Level

The `Serializable` isolation level provides the strictest transaction isolation. 
This level emulates serial transaction execution, as if transactions had been executed one after another, serially, rather than concurrently. 
However, like the Repeatable Read level, applications using this level must be prepared to retry transactions due to serialization failures. 

## Links

More details can be taken here: [PostgreSQL Documentation. Chapter 13. Concurrency Control. 13.2. Transaction Isolation](https://www.postgresql.org/docs/9.2/static/transaction-iso.html)


