+++
date = "2017-12-04T16:14:52+03:00"
draft = false
title = "Optimistic and pessimistic concurrency control"
tags = ["database"]
+++

Transactional isolation is usually implemented by locking whatever is accessed in a transaction. 
There are two different approaches to transactional locking: 

* Pessimistic locking
* Optimistic locking

## Pessimistic concurrency control (locking)

Pessimistic locking is called "pessimistic" because the system assumes the worst — 
it assumes that two or more users will want to update the same record at the same time, 
and then prevents that possibility by locking the record, no matter how unlikely conflicts actually are.

Pessimistic locking assumes that data will be changed by another transaction and so locks it.
Pessimistic locking locks the records as soon as it selects rows to update. 
The pessimistic locking strategy guarantees the changes are made safely and consistently.

The disadvantage of pessimistic locking is that a resource is locked from the time it is first accessed in a transaction 
until the transaction is finished, making it inaccessible to other transactions during that time. 
If most transactions simply look at the resource and never change it, an exclusive lock may be overkill 
as it may cause lock contention, and optimistic locking may be a better approach.

## Optimistic concurrency control (locking)

Optimistic locking assumes that although conflicts are possible, they will be very rare. 
Instead of locking every record every time that it is used, the system merely looks for indications 
that two users actually did try to update the same record at the same time. 

Optimistic locking locks the record only when updating takes place. 
It ensures that the locks are held between selecting, updating, or deleting rows. 
This process needs a way to ensure that the changes to data are not performed between the time of being read and being altered. 
This is achieved using version numbers, timestamps, hashing, etc.

The primary advantage of optimistic locking is , it minimizes the time for which a given resource is unavailable 
which is used by another transaction and in this way it more scalable locking alternative.

## What to use - optimistic or pessimistic concurrency control?

In most scenarios, optimistic concurrency control is more efficient and offers higher performance. 
When choosing between pessimistic and optimistic locking, consider the following:

* Pessimistic locking is useful if there are a lot of updates and relatively high chances of users trying to update data at the same time.
* Pessimistic concurrency control is also more appropriate in applications that contain small tables that are frequently updated.
* Optimistic locking is useful if the possibility for conflicts is very low – there are many records but relatively few users, or very few updates and mostly read-type operations.

So, there is no correct answer – it depends. You should pick your locking scheme based on your application requirements.

## Links

[Wikipedia - Optimistic concurrency control](https://en.wikipedia.org/wiki/Optimistic_concurrency_control)

[IBM SolidDB Guide - PESSIMISTIC vs. OPTIMISTIC concurrency control](https://www.ibm.com/support/knowledgecenter/en/SSPK3V_7.0.0/com.ibm.swg.im.soliddb.sql.doc/doc/pessimistic.vs.optimistic.concurrency.control.html)

[ObjectDB Manual - Locking in JPA](http://www.objectdb.com/java/jpa/persistence/lock)