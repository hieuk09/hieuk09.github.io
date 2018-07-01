---
layout: post
title: "Reading note: High Performance MySQL - Indexing Strategies"
date:   2018-06-26
categories: database
---

There are many ways to use indexes. All of them are optimized for a few special
cases. Therefore, it's better to understand and benchmark them before applying.

## Isolate Column

- A column must be isolated in a query in order to be used with indexes.
- Advice: Simplify your query, so that a column always be in one side of the
criteria

## Prefix indexes

- We can reduce the size of indexes of string columns by indexing only prefix of
the column. However, using only a part of the column can also decrease selectivity, which
leads to less efficient index. Therefore, we must balance between size and
selectivity.

## Clustered indexes

- Only a few storage engines support this indexes (like InnoDB). Most of other
storage engines (like MyISAM) don't support it.
