---
layout: post
title: "Book review - High Performance MySQL"
date:   2018-09-02
categories: book database
---

# Info

![book](http://{{ site.url }}/assets/high_performance_mysql.png)

```
- Pages: 2025
- Author:  Baron Schwartz, Peter Zaitsev, Vadim Tkachenko
- Publisher: O'Reilly
```

# Review:

High Performance MySQL is a great book which talk not only about MySQL,
but also database in general. It shows you the ways to make your MySQL
database more performance, and help you understand why and how it can be more
performance in those ways. Along the way, it explains a lot of database concepts,
which helps understanding other databases as well.

However, some of the information in the book is outdated (the third version only
covers up to MySQL 5.5), so you should probably check your current version manual
to make sure about some MySQL functionality.

# Table of contents:

> Note:
> - Normal text: can be skipped
> - *Italic text*: should at least skim through
> - **Bold text**: should read carefully

- Preface: This part is about the overview of the book, you can skim through it
to get an idea of what this book is about.

- **Chapter 1** - MySQL architecture: This is an important chapter, as the title
say, it talks a about what MySQL architecture is. It also talks about some
concepts in MySQL like concurrency, locks, transaction, isolation level and
storage engines. I think these information are great if you want to understand
other database systems as well.

- *Chapter 2* - Finding Bottlenecks: Benchmarking and Profiling:
talks about Benchmarking and Profiling strategies and tools. It shows some
insights about benchmarking and profiling in general, and the ideas can apply
anywhere, not just for MySQL database. This part is not critical, but if you
have time, you should at least skim through it.

- **Chapter 3** - Schema Optimization and Indexing: talks about the
most important topic of database optimization - indexing. It describes how MySQL
implements several types of index and how to use them correctly with your
application requirement.

- **Chapter 4** - Query Performance Optimization: describes how the
query optimizer works. It also mentions several techniques to structure your
query and schema to help the optimizer optimize your query better.

- *Chapter 5* - Advance MySQL features: as the name, talks about advance
features in MySQL, which includes Query Cache, Views, Charset, Full-text search, Store
Proceduces, ...

- *Chapter 6* - Optimizing Server Setting: shows how MySQL settings
can affect your query performance, and how to optimize them. Most of these
requires that you have a good set of benchmark for your database, so you should
read Chapter 2 before this.

- Chapter 7 - Operating System and Hardware Optimization: Unless you need to
install MySQL in a separate machine instead of using RDS (or Aurora), you
probably don't need this chapter. However, it's still worth reading just in case
you ever need to optimize those settings.

- *Chapter 8* - Replication: Another important topic for maintaining a database
system. Aside from replicating with MySQL, the book also describes how to
replicate the whole filesystem or server, which is sometimes easier and simpler.

- *Chapter 9* - Scaling and High Availability: discusses several
techniques to scale: scale up, scale out, partition, sharing, load balance, ...

- *Chapter 10* - Application-level Optimization: explains in-depth about caching,
  as well as discussing about other techniques such as compression, trimming
  down unused modules, ...

- *Chapter 11* - Backup and Recovery: as I never administrates any systems, I
never know about how important backup and recovery are. A take away from this
chapter: always have backup and recovery process, test them regularly.

- *Chapter 12* - Security: talks about how MySQL handles permissions and
priviliges of users. It also discusses some bad practices that should be avoided
when setting up MySQL permissions.

- *Chapter 13* - MySQL Server status: shows a lot of information about the
metrics we can get when monitoring MySQL Server.

- Chapter 14 - Tools for high performance: shows several tools for maintaining
MySQL system. Personally, I think the list is pretty outdated and we can find
better alternatives on the internet.

- Apendix A - Transfering Large Files

- Apendix B - Using EXPLAIN

- Apendix C - Using Sphinx with MySQL

- Apendix D - Debugging Locks
