+++
date = "2017-11-28T07:14:59+03:00"
draft = false
title = "Transaction isolation levels"
tags = ["database", "jdbc"]
+++

Let's speak about transaction isolation levels. 

## ANSI/SQL standard isolation levels

We have 4 standard-defined transaction isolation levels:

* **Read uncommited.** Dirty reads - YES, Non-Repeatable reads - YES, Phantom reads - YES.
* **Read commited.** Dirty reads - NO, Non-Repeatable reads - YES, Phantom reads - YES.
* **Repeatable read.** Dirty reads - NO, Non-Repeatable reads - NO, Phantom reads - YES.
* **Serializable.** Dirty reads - NO, Non-Repeatable reads - NO, Phantom reads - NO.

Here `YES` means `may occur`, while `NO` means `should not occur at all`. So `NO` here is more strong expression than `YES`. 
Different DBMS may make this `YES` more restrictive. E.g. PostgreSQL does not allow `Phantom Reads` on its `Repeatable read` level. 
See details here: [PostgreSQL transaction isolation levels](/post/postgresql_transaction_isolation_levels/).

## Read phenomena

The ANSI/ISO standard SQL 92 refers to three different read phenomena when Transaction 1 reads data 
that Transaction 2 might have changed:

* Dirty reads
* Non-Repeatable reads
* Phantom reads

Let's consider them. Let's operate with following database table **users** that has following data.

| id | name | age |
|----|------|-----|
| 1  | John | 20  |
| 2  | Eva  | 30  |


## Dirty reads

A *dirty read* (aka uncommitted dependency) occurs when a transaction is allowed to read data from a row 
that has been modified by another running transaction and not yet committed.

|Transaction 1                           |Transaction 2                                       |
|----------------------------------------|----------------------------------------------------|
| `/* Query 1. Will read 20*/`           |                                                    |
| `select age from users where id=1`     |                                                    |
|                                        | `/* Query 2. No commit here. */`                   |
|                                        | `update users set age=50 where id=1`               |
| `/* Query 1. Will read 50 */`          |                                                    |
| `select age from users where id=1`     |                                                    |
|                                        | `/* Lock-based dirty read. */`                     |
|                                        | `rollback`

We can see here that *dirty read* will read age value of `50` in the 2nd query, even if *Transaction 2* will be rolled back afterwards. 
 
## Non-Repeatable reads

A *non-repeatable read* occurs, when during the course of a transaction, 
a row is retrieved twice and the values within the row differ between reads.

|Transaction 1                           |Transaction 2                                       |
|----------------------------------------|----------------------------------------------------|
| `/* Query 1. */`                       |                                                    |
| `select * from users where id=1`       |                                                    |
|                                        | `/* Query 2. */`                                   |
|                                        | `update users set age = 50 where id = 1;`          |
|                                        | `commit;`                                          |
| `/* Query 1. */`                       |                                                    |
| `select * from users where id=1;`      |                                                    |
| `commit;`                              |                                                    |

During *Transaction 1* *Transaction 2* is started and committed, and it changes rows that should be returned as a result 
of *Transaction 1*. But *Transaction 1* has already seen the different value before *Transaction 2* starts. 
So what value should it return? 

If transaction isolation level is set to either to `Repeatable read` or `Serializable` (levels at which Non-Repeatable reads
do not occur) - then *Transaction 1* should return old values (where age is 20). 

If transaction isolation level is set to either to `Read uncommited` or `Read commited` (levels at which Non-Repeatable reads
may occur) - then *Transaction 1* may return new values (which incorporate changes made by *Transaction 2*, i.e. where age is 20). 

## Phantom reads

A *phantom read* occurs when, in the course of a transaction, two identical queries are executed, 
and the collection of rows returned by the second query is different from the first.

|Transaction 1                        |Transaction 2                              |
|-------------------------------------|-------------------------------------------|
| `/* Query 1. */`                    |                                           |
| `select * from users `              |                                           |
| `where age between 10 and 30;`      |                                           |
|                                     | `/* Query 2. */`                          |
|                                     | `insert into users(id,name,age)`          |
|                                     | `values (3, 'Bob', 27); commit;`          |
| `/* Query 1. */`                    |                                           |
| `select * from users `              |                                           |
| `where age between 10 and 30;`      |                                           |

At the highest isolation level (`Serializable`) *Transaction 1* will return the same set of rows 
(i.e. it will return 2 rows). While at lower transaction isolation levels (`Uncommited read`, `Commited read`, `Non-repeatable read`)
it will return set of rows including those rows added with *Transaction 2*, i.e. it will return 3 rows. 

## JDBC 

JDBC driver can have following transaction isolation levels: 

* `Connection.TRANSACTION_NONE` - a constant indicating that transactions are not supported
* `Connection.TRANSACTION_READ_UNCOMMITTED` - read uncommited level
* `Connection.TRANSACTION_READ_COMMITTED` - read commited level
* `Connection.TRANSACTION_REPEATABLE_READ` - repeztable read level 
* `Connection.TRANSACTION_SERIALIZABLE` - serializable level

See more details here: [java.sql.Connection](https://docs.oracle.com/javase/8/docs/api/java/sql/Connection.html).