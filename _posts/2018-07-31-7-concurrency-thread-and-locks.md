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
