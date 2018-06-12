---
layout: post
title: "Reading note: High Performance MySQL - B-Tree index"
date:   2018-06-12
categories: database
---

- Most of MySQL storage engines support this index except Archive Storage
Engine.

- MySQL uses the same indicator `BTREE` in `CREATE TABLE` statement but the
storage engines implement this index differently.

- General idea:
  + Similar to a binary tree
  + All values are stored in order
  + Each leaf page has the same distance to the root

- Use-case:
  + Searching for ranges of data
  + Searching for single record

- Applicable for search type:
  + Match full value
  + Match left most prefix
  + Match column prefix
  + Match range of values
  + Match one part exactly and match prefix in another part
  + Index only queries

- Limitation: Cannot fully apply for:
  + Queries does not start from left most side
  + Queries that skip column. Ex: index for 3 columns (first_name, last_name and
      dob) but only query first_name and dob.
  + Queries that use range in the middle

- Using hash index to improve index on column with big data:
  + Use a simple hash function like `CRC32` to create a column from original
  column
  + Index new column with hash index
  + Query new column with original column to improve performance
