---
layout: post
title: "Reading note: High Performance MySQL - Benchmarking strategies and
tactics"
date:   2018-05-29
categories: database
---

## Why benchmarking?

- Measure current system performance
- Validate scalability: by using load much higher than reality
- Foresee necessary resource for grow
- Validate improvement from upgrade/change

## Strategies

There are two common strategies:

- Full-stack benchmark: benchmark the whole application
- Single-component benchmark: benchmark only MySQL

### Full-stack benchmark

Reason:

- Testing whole system capability and performance
- MySQL may not always be the bottle-neck
- Testing real system behavior and how each component interacts with each other

Difficulty:

- Hard to setup correctly
- Result may not reflect reality

### Single-component benchmark:

Reason:

- Validating micro-optimization, such as changing query or schema
- Benchmark a specific problem in the application
- Get faster cycle of feedback for changes

Difficulty:

- Should use real dataset instead of fake one
- Result may not reflect reality

## What to measure

- Transaction per time unit: also called *throughput*, usually used for OLTP
applications, most suitable for multiuser interactive applications. Usually use
second as unit.

- Response time: also called *latency*, measures total time for a task, from
that, derives average/minimum/maximum time, then derives *percentile response
time*. A graph is very helpful to show this, rather than just number.

- Scalability: performance under a workload, good for capacity planning

- Concurrency: how many requests/sec users can generate at peak time. This
metric is also affected by language, toolkit and framework, not just database.

## Tactics

Common mistakes:

- Use only a subset of real data size
- Use incorrect distributed data (like random data)
- Use unrealistically distributed parameters (like view all data equally)
- Use single-user scenario for multi-user scenario
- Benchmark distributed application on single server
- Fail to match real user behavior (such as thinking time)
- Run identical queries in a loop, which is cached
- Fail to check error
- Ignore warm up
- Use default settings

### Designing and planning

Steps:

- Identify problem and goal
- Choose benchmark that matches your need (don't use TPC for e-commerce
application)
- Take database snapshot
- Create queries to run against data
- Log these queries
- Write down benchmark plan
- Document parameters & result

### Getting accurate results

- Design benchmark for your need
- Make sure benchmark result is repeatable
- Watch out for external load, profiling and monitoring system, and any other
factors that can skew your results
- Isolate variable (change only a few parameters for each benchmark)
- Investigate the result and figure out what happen

### Running and analyzing results

- Automate benchmark run
- Run the benchmark many times
- Apply scoring system: can be statistical model or average model
