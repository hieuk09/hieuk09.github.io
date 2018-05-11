---
layout: post
title: "Reading note: High Performance MySQL - Concurrency Control"
date:   2018-05-12
categories: database
---

## Locks

MySQL offers two kinds of lock:

- **Read lock**: Lock when reading, does not block other reads. Therefore, also
called *shared lock*

- **Write lock**: Lock when writing, blocks other reads and writes. Has higher
priority than read lock. Also called *exclusive lock*

## Lock granularity

Two most popular locking strategy:

- **Table locks**: most basic type of locking, has low overhead, but low
concurrency, too. Locks the entire table. Offered by both MySQL server
and storage engines. Usually used in statement like `ALTER TABLE`.

- **Row locks**: locking type that offers highest level of concurrency, but also the
highest overhead. Available in InnoDB and Falcon storage engine. MySQL server
does not aware of these locks.

## Isolation Level

MySQL defines 4 isolation levels:

- **READ UNCOMITTED**: transaction can view results of uncommitted transactions.
Good performance. Not recommended.

- **READ COMMITTED**: transactions cannot view results of uncommited transactions.
However, *nonrepeatable read* still happens. Good performance.

- **REPEATABLE READ**: default settings of MySQL. Prevent *nonrepeatable read*. Ok
performance. *phantom read* can still occur. InnoDB and Falcon prevent phantom
read by using multiversion concurrency control, though.

- **SERIALIZABLE**: highest isolation level. Prevent all concurency issues by
forcing transactions to be ordered so they cannot cause conflict. Poor
performance dues to a lot of lock usage. Recommended if data stability is crucial.

## Transaction Logging

- Use write-ahead logging

- Recovery method is storage engine dependent

## Autocommit

- When enabled: each query is in separate transactions.

- When disabled: every query is in a transaction until a COMMIT or ROLLBACK.
After that, a new transaction is started.

- This setting affects only transactional storage engine. Nontransactional
storage engines like MyISAM is not affected because they are always on
AUTOCOMMIT mode.

- It can be enabled/disabled using variable `AUTOCOMMIT`

## Mixing storage engines

- Cannot reliably mix storage engine in a transaction

- When a ROLLBACK occur in a transaction has mixed storage engine of
nontransactional engine and transactional engine, the data in nontransactional
engine is not reverted.

- MySQL does not offer any warnings most of the time

## Implicit and Explicit locking

- InnoDB uses two-phase locking protocol, usually with implicit locking.

- InnoDB also supports explict locking with `LOCK IN SHARE MODE` and `FOR
UPDATE`

- MySQL supports explicit locking with `LOCK TABLES` and `UNLOCK TABLES`
commands

