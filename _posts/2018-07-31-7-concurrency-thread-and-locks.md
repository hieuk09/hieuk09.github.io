---
layout: post
title: "Reading note: 7 concurrency models in 7 weeks - Threads and Locks"
date:   2018-07-31
categories: concurrency
---

## Day 1

- Problem: race condition when accessing shared memory, JVM/compiler/hardware
optimize the code, causing change in logic for multiple-threads-program
- Solution: instrinsic lock - `synchronized`, `volatile` keyword
- Problem with instrinsic lock: dead lock
- Solution: Acquire locks in a fixed, global order, do not call alien methods,
  hold lock for short amount of time.
- Read more: [Java memory model](http://www.cs.umd.edu/~pugh/java/memoryModel/jsr-133-faq.html)

## Day 2

- Problem: the need of thread interruption, timeout when acquiring lock, control acquiring
and releasing lock
- Solution: Reentrant Lock, Atomic variables
- Read more:
  + [Fairness in Reentrant Lock](http://tutorials.jenkov.com/java-concurrency/starvation-and-fairness.html)
  + [ReentrantReadWriteLock](http://www.byteslounge.com/tutorials/read-write-locks-in-java-reentrantreadwritelock)
    - The lock is separated into `read_lock` and `write_lock`. This is for
    optimizing for workload with a lot of read and sporadic write.
    - Read and Write lock can also have fairness and timeout
    - Read -> Write is upgraded, and Write -> Read is downgraded
  + [Spurious Wakup](http://blog.vladimirprus.com/2005/07/spurious-wakeups.html):
  the event when a call to `wait` returns even if the condition variable does not update, which forces developers
  to check the condition variable in a while loop. A good design software does
  not need to care about this because it uses better locking mechanisim instead
  of condition variable.
  + [AtomicReferenceFieldUpdater](http://normanmaurer.me/blog/2013/10/28/Lesser-known-concurrent-classes-Part-1/)
