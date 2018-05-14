---
layout: post
title: "Reading note: High Performance MySQL - MVCC"
date:   2018-05-12
categories: database
---

Multiversion Concurrency Control (MVCC) is a technique used by MySQL to reduce
locking usage in isolation level like **Read Committed** and **Repeatable
Read**.

**Example 1**: let's consider a common use-case:

- User A has a balance of 400$ and wants to transfer 100$ to user B (balance
300$)

- At the same time, user C views user A and user B account

The SQL command can be something like:

```SQL
BEGIN TRANSACTION
  SET balance = balance - 100 FROM users WHERE users.name = 'A'; # (1)
  SET balance = balance + 100 FROM users WHERE users.name = 'B'; # (2)
END

# between (1) & (2)

BEGIN TRANSACTION
  SELECT name, balance FROM users WHERE users.name IN ('A', 'B'); # (3)
END
```

## Read uncommitted isolation level

(3) can return something like:

```
name | balance
---------------
A    | 300
B    | 300
```

Why? Because when (1) is finished, the database write the changes, so (3) can
read it. This may cause some confusion when user C see that 100$ just vanish into thin
air, so we want to avoid that.

The issue above is called **Dirty read** or **Read uncommitted**, and usually is
prevented in **Read committed** isolation level.

## Read committed isolation level

One solution is to use lock. After (1), the record of user A acquires a write lock.
When (3) run, the database notices the lock, so the query is put to a queue. After (2),
the lock is released, and (3) is taken from the queue and run.

Now, (3) can return something like:

```
users | balance
---------------
A     | 300
B     | 400
```

However, write locks can block all read queries. If your application has a
lot of read, a slow write can block all the reads, causing the whole application
to be slow down.

To prevent that, we store the row with a committed flag. The read query will
not read this row if committed is false.

Back to the example:

After (1), the latest user A row has committed as false, so when (3) run, it
returns the committed version:

```
users | balance
---------------
A     | 400
B     | 300
```

Read query in this case is not block by write query, so it can perform with good
latency even if there are slow write queries. However, storing a new attribute
and multiple versions of each row has its own overhead.

Let's move to another example:

**Example 2**: your mailbox shows that there is one unread mail when there is
a new mail. The query when a mail arrives is:

```SQL
BEGIN TRANSACTION
  INSERT INTO mails (content, read) VALUE (content, FALSE); # (1)
  SET notifications_count =
    (
      SELECT COUNT(*) FROM mails WHERE read IS FALSE
    )
  FROM mailboxes; # (2)
END

BEGIN TRANSACTION
  SELECT notifications_count FROM mailboxes; # (3)
  SELECT * FROM mails WHERE read IS FALSE; # (4)
END
```

If you only have committed and uncommitted versions of the row, even if the
queries ordered is sequencial like (1) -> (2) -> (3) -> (4), the result can
be:

```
notications_count
-----------------
0
```

Why? Because in (2), the new row of mails is uncommitted, so it isn't counted
into the notifications count. We must have a way to know that the uncommitted
row belongs to the current transaction. One way to do it is using a
monotonically increased number as transaction identifier (transaction_id).

We assign the transaction_id to all uncommitted records in the transaction, and
allow read query to read committed row and uncommitted row with same
transaction_id.

The example now returns correctly:

```
notications_count
-----------------
1
```

However, if query (1) & (2) is concurrent with (3) & (4), the order may become
(3) -> (1) -> (2) -> (4). In that case, the query may returns:

```
notications_count
-----------------
0

content     | read
-------------------
new content | false
```

Why? Because when (3) finishes, notifications_count has not been updated yet,
but when (4) finishes, the new mail has been inserted to database and is
committed.

This issue is called **Nonrepeatable read** or **Read skew**, and is prevented
in **Repeatable read** isolation level.

## Repeatable Read isolation level

This isolation level uses MVCC technique, which is very similar to the way we
store `transaction_id` above. Instead of a single `transaction_id`, it uses
`created_id` and `deleted_id` to store `transaction_id`.

- When a record is created (using `INSERT`), it has `created_id` as the current
transaction_id and `deleted_id` as `nil`.

- When a record is updated, a new row is created with `created_id` as the
current transaction_id and `deleted_id` as `nil`. The old row is updated with
`deleted_id` as the current transaction_id.

- When a record is deleted, `deleted_id` of the row is updated with current
transaction_id.

- When reading, we filter all versions of the row using 2 criterias:
  + row must have `created_id` <= current transaction id
  + row must have `deleted_id` > current transaction id or `nil`

Back to the example 2, if query runs with order (3) -> (1) -> (2) -> (4), we see
that the result is:

```
notications_count
-----------------
0

content     | read
-------------------
```

If query runs with order (1) -> (3) -> (2) -> (4), we'll see:


```
notications_count
-----------------
0

content     | read
-------------------
```

If query runs with order (1) -> (2) -> (3) -> (4), we'll see:

```
notications_count
-----------------
1

content     | read
-------------------
new content | false
```

Why? Let's look at (3) -> (1) -> (2) -> (4)

```
(3) => notifications_count = 0
       transaction_id = 0
       created_id = 0
       deleted_id = nil

(1) => notifications_count = 1
       transaction_id = 1
       created_id = 1
       deleted_id = nil

(2) => new mail record
       transaction_id = 1
       created_id = 1
       deleted_id = nil

(4) => no mail
       transaction_id = 0
```

We see that because new mail record has `created_id` as `1`, so (4) with
transaction_id as 0 cannot query it.

However, this technique cannot fully prevent issue like **phantom read**.

InnoDB uses `created_id` similarly to how `transaction_id` is used in the
**Read Committed** section here.

## Compatibility

MVCC is used in **Read committed** and **Repeatable Read** only, so **Read
Uncommitted** and **Serializable** are incompatible with them.
