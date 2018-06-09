---
layout: post
title: "Reading note: High Performance MySQL - Choose optimal types"
date:   2018-06-06
categories: database
---

## Things to remember

- Smaller is usually better: smaller type takes less space to store, which leads
to less memory consumption, more likely to cache and faster to process.

- Simpler is better: Simple type uses less CPU cycles, so usually is faster to
processed.

- Avoid `NULL` value if possible: `NULL` value adds more complexity to
calculation when querying and processing a value. Avoid this overhead by using
`NOT NULL` for column that does not have NULL value or using a special value
instead. Performance impact of this is not big, though.

### Example:

- If your database has to store data about all countries, you don't have to use
`BIGINT` as identifiers of the countries. Usually, a `TINYINT` or `SMALLINT`
column is enough.

- Use integer for IP address instead of string: IP address is a 4-bytes or 6-bytes integer,
so using integer saves a lot of space, has much faster `SELECT` query, and
is easier to manipulate

- MyISAM index on nullable column can cause a fixed-size index to be converted
to a variable-sized index

## Steps to choose type

- Choose general type: is it a string, number, temporal or custom type?
- Choose specific type: If it is a number, is it a `INT` or `DOUBLE`? If it is
time, should it be `TIMESTAMP` or `DATETIME`?

## Whole Number

```
type name | TINYINT | SMALLINT | MEDIUMINT | INT    | BIGINT
------------------------------------------------------------
size      | 8 bit   | 16 bit   | 32 bit    | 32 bit | 64 bit
```

- `UNSIGNED` attribute is optional.
- Choice affects how MySQL stores data, not how it computes. Computing usually
uses 64-bit integer.
- Specifying width such as `INT(11)` is meaningless on storage level, it only
affects how value is represented in CLI.

## Real Number

- Decimal
  + Can store whole number that does not fit in `BIGINT`
  + Can have maximum 65 bit in MySQL 5.0 to newer
  + Is exact fractional numbers from MySQL 5.0 to newer
- Float and Double
  + Float is 4-bytes and Double is 8-bytes
  + Is inexact fractional number

## String

- For relative small strings:
  + `VARCHAR(100)` is much slower than `VARCHAR(5)` for operation that requires
a lot of memory or sort. Therefore, allocate only the maximum space that you
need.

```
type name |  VARCHAR                           | CHAR
---------------------------------------------------------------------------------------
attribute | - variable-length string           | - fixed length string
          | - preserves trailing white spaces  | - removes trailing while spaces
---------------------------------------------------------------------------------------
usage     | - maximum length is much bigger    | - fixed-size column like password hash
          | than average length                |
```

- For big strings: `BLOB` and `TEXT`
  + stores in "external" storage area
  + `BLOB` stores binary data while `TEXT` stores text with collation and
  character set
  + orders by only the first `max_sort_length`
  + cannot index the full length of these types

- For dictionary-type: ENUM
  + stores with an integer key and a value in a lookup table
  + sorts by integer key
  + list of strings is fixed and requires `ALTER TABLE` to modify
  + has some overhead when join with `VARCHAR` or `CHAR` column

## Datetime

- MySQL storages time with one-second granularity
- However, it can do temporal computational with microsecond granularity

```
type name | DATETIME                           | TIMESTAMP
----------------------------------------------------------------------------------------
attribute | - ranges from year 1001 to year    | - ranges from midnight 01/01/1970 (GMT)
          | 9999.                              | to 2038
          | - stores as an integer in          | - stores the seconds from the start
          | YYYYMMDDHH-MMSS format             | time as an integer
          | - value is timezone independent    | - value is timezone dependent
```

## Bit-packed Data type

- `BIT`:
  + Is used to store many boolean value in a single column
  + Has different behaviors in different storage engines
  + Has maximum 64 bit
  + Is treated as string type

- `SET`:
  + Combines many boolean value into a single data structure
  + Is stored efficiently
  + Has `FIND_IN_SET` and `FIELD` function to query
  + Does not support index
  + Changing column definition is very expensive

- Bitwise operation in integer column:
  + Uses an integer to store boolean value
  + Manipulates value using bitwise operation
  + May be harder to maintain

## Choosing identifiers

- Same principle as choosing normal type, plus: use the same type (even `UNSIGNED`
attribute) for foreign key columns in related table

```
type name   | Integers              | Set or Enum           | String
----------------------------------------------------------------------------------
Performance | Good                  | Good                  | Bad
----------------------------------------------------------------------------------
Simplicity  | Good - has            | Bad - No              | Normal - randomize
            | AUTO-INCREMENT        | AUTO-INCREMENT        |
----------------------------------------------------------------------------------
Flexibility | Good                  | Bad - should be used  | Good
            |                       | only for static table |
```
