---
layout: post
title: "Reading note: High Performance MySQL - Table conversion"
date:   2018-05-20
categories: database
---

## Table conversion

There are multiple ways to convert a table from one storage engine to another.
Each way has each own advantages and disadvantages.

## Alter Table

- Use `ALTER TABLE` statement
- Can take a lot of time
- Use a lot of disk I/O and original table will be read-lock
- Storage-engine specific feature will be lost

## Dump and Import

- Dump data using `mysqldump` utility and import it back
- Must change table name and type to take effect
- Dump file has `DROP TABLE` on top, so be careful

## Create and Select

- Use `INSERT ... SELECT ...` statement
- A compromise between speed and safety compared to two above options
- Can run in chunk instead of the whole table
