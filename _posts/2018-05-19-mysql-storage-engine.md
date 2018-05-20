---
layout: post
title: "Reading note: High Performance MySQL - Storage Engine"
date:   2018-05-19
categories: database
---

## Storage Engine

- MySQL storage engine is a separate layer from MySQL server.
- Each table can use different storage engine.
- However, MySQL server manages table definition, and its name is dependent on
platform: Windows allows case-insensitive name, and Linux is case-sensitive.

We'll look through a few popular engines:

## MyISAM Engine

- One of the oldest storage engine of MySQL
- Has separate data and index file. The max size is the max file size allowed by
OS.
- Supports advance index features: GIS, full-text index
- Is non-transactional
- Has auto/manual repair
- Has fast insert/read queries; however, modify queries lock entire table,
  blocks all read queries.
- Has delay key-write to improve performance when writing index
- Has compress mode for CD-ROM and DVD-ROM: has small size and fast read due to
low disk I/O

## MyISAM Merge Engine

- Merges multiple MyISAM tables to a virtual table
- Usually uses in datawharehousing

## InnoDB Engine

- One of the most popular engines
- Is built for short-live transactions
- Is transactional, and implements 4 MySQL isolation levels.
- Uses MVCC for high concurrency
- Default isolation level is Repeatable Read, which uses next-key locking to
prevent phantom read.
- Has multiple data files, which separate table data and indexes.
- Has clustered indexes
- Fast primary key lookup
- Secondary indexes also has primary key, so indexes for table with a lot of
records can be very big and slow.
- Has slower table rebuild time compared to MyISAM
- Implements Foreign key constraint, which MySQL does not implement.

## Memory Engine

- Isn't persisted to disk
- Is faster than MyISAM
- Supports Hash index
- Supports only fix-size row
- Uses table-lock for each read/write, so it has low concurrency
- Is usually used for cache, lookup query and intermediate result

## Archive Engine

- Supports only INSERT and SELECT queries
- Has much less disk I/O than MyISAM because it buffers data writes and
compresses row with zlib when inserting
- Each SELECT query requires full table scan
- Is usually used for logging and data acquisition
- Supports row-level locking and special buffer system for high-concurrency
insert
- Is non-transactional

## CSV Engine

- Can treat csv file as tables
- Lets user copy file in/out when server is running
- Is usually used for data-interchange

## Federated Engine

- Does not store data locally, but uses a table on a remote MySQL server instead
- Has very slow aggregate queries, joins or other basic operations
- Is only useful for single-row lookup or INSERT queries on a remote server

# Blackhole Engine

- Has no storage mechanism
- Discards all INSERT queries, but keep the logs as usual
- Is useful for replication setups and audit logging

# NDB Cluster Engine

- Is designed for high performance with redundancy and load-balancing
capabilities
- Keeps all data in memory and is optimized for primary key lookups
- Has share-nothing infrastructure, consists of data nodes, management nodes and
SQL nodes (MySQL instances)
- Each data nodes is a shard of the cluster's data, which is replicated to
multiple nodes
- Performs joins at MySQL server level, and can be extremely slow due to network
latency. On another hand, single-table lookups can be very fast.
- Is very large and complex
- Is usually unsuitable for traditional applications

## solidDB Engine

- Is transactional
- Uses MVCC and supports both pessimistic and optimistic concurrency control
- Has full foreign key supports

## PBXT Engine

- Uses transaction logs and data files to avoid write-ahead logging, which
reduces overhead
- Uses MVCC
- Supports foreign key constraint
- Does not use clustered index
- Has BLOB streaming: when combining with BLOB engine, it can stream binary and
media file directly in and out of the database

## Maria Engine

- Now is called Aria in MariaDB
- Replaces MyISAM as default engine of MariaDB
- Uses Aria table format instead of MyISAM, which speeds up some GROUP BY and
DISTINCT queries

## Selecting right Engine

You should take into account a few factors:

- Transaction
- Concurrency
- Backup
- Crash recovery
- Special features: foreign keys, joins, indexes
