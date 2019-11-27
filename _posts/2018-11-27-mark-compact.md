---
layout: post
title: "Mark-Compact algorithm"
date:   2019-11-28
categories: book
---

On Oct 22nd, Ruby 2.7 preview2 was released. One of the notable feature is
the ability to compact memory using GC.compact. So what does it do exactly?

## Revisit Ruby GC

Ruby GC currently uses an incremental generational mark-sweep algorithm.
A mark-sweep algorithm is basically: for each GC iteration, GC loop through the memory twice.

- In the first pass, GC loops through the roots, traces their references, then marks the objects as live.

```
                         Garbage
                            |
                            v
 -------------------------------------
 | X X | X X |     |     | X X | X X |
 |  X  |  X  |  +  |  +  |  X  |  X  |
 |_X_X_|_X_X_|_____|_____|_X_X_|_X_X_|
                ^
                |
           Live Object
```

- In the second pass, GC frees up memory that the garbage objects occupied,
  leaving on marked object behind

```
                         Free memory
                            |
                            v
 -------------------------------------
 |     |     |     |     |     |     |
 |     |     |  +  |  +  |     |     |
 |_____|_____|_____|_____|_____|_____|
                ^
                |
           Live Object
```

The problem with this algorithm is that now the memory is fragmented,
which can be bad in the long run: even when there is memory available, they cannot be allocated.

## Mark-compact algorithm

Mark-compact algorithm can eliminate fragmentation in order to have a compacted heap.
The major benefit of a compacted heap is that allocation can be very fast,
and objects that are adjacent can be benefit from CPU cache.

The idea of a mark-compact algorithm is pretty similar to mark-sweep algorithm,
with the first step is almost the same: trace through objects, then mark them as live.
The second step is:

- Move the live objects to the available memory slots
- Update the references to point to new objects' location

```
                             Garbage
                                |
                    _______     |
                    |     |     |
                    v     |     v
     -------------------------------------
     | X X | X X |     |     | X X | X X |
     |  X  |  X  |  1  |  2  |  X  |  X  |
     |_X_X_|_X_X_|_____|_____|_X_X_|_X_X_|
                    ^
                    |
               Live Object
```

```

                             Garbage
      reference pointer         |
              _____________     |
              |     |     |     |
              |     v     |     v
     -------------------------------------
     |     |     |     |     | X X | X X |
     |  1  |  2  |  1* |  2* |  X  |  X  |
     |_____|_____|_____|_____|_X_X_|_X_X_|
         ^          |
         |__________| forward pointer
         |
    Live Object
```

```
                             Free space
      reference pointer         |
        _______                 |
        |     |                 |
        v     |                 v
     -------------------------------------
     |     |     |     |     |     |     |
     |  1  |  2  |     |     |     |     |
     |_____|_____|_____|_____|_____|_____|
         ^
         |
         |
    Live Object
```

### "Two finger" compaction

This is the simplest compaction algorithm, best suite
for compacting memory region with object of a **fixed size**.
The idea is that given the column of live data in a region,
we know a threshold that all data can be fixed in.
We can move all live objects above the threshold to the gap below the threshold.

There are two pointer `free` and `scan` (which are "two fingers"),
we advance `free` pointer each time we encounter a live object,
and `scan` is moved back each time we encounter a garbage or free memory slot.
When `free` encounters a free memory slot and `scan` encounters a live object,
the live object is moved to `free` slot.

```
     -------------------------------------------------------------------
     |     |     | X X | X X | X X |     | X X |     |     |     |     |
     |  +  |  +  |  X  |  X  |  X  |  +  |  X  |  +  |  +  |  +  |     |
     |_____|_____|_X_X_|_X_X_|_X_X_|_____|_X_X_|_____|_____|_____|_____|
     ^                                   |                             ^
     |                                   |                             |
     |                                  threshold                      |
     free                                                            scan
```

```
     -------------------------------------------------------------------
     |     |     | X X | X X | X X |     | X X |     |     |     |     |
     |  +  |  +  |  X  |  X  |  X  |  +  |  X  |  +  |  +  |  +  |     |
     |_____|_____|_X_X_|_X_X_|_X_X_|_____|_X_X_|_____|_____|_____|_____|
                 ^                                                     ^
                 |                                                     |
                 |                                                     |
                 free                                                scan
```

```
     -------------------------------------------------------------------
     |     |     | X X | X X | X X |     | X X |     |     |     |     |
     |  +  |  +  |  X  |  X  |  X  |  +  |  X  |  +  |  +  |  +  |     |
     |_____|_____|_X_X_|_X_X_|_X_X_|_____|_X_X_|_____|_____|_____|_____|
                 ^                                               ^
                 |                                               |
                 |                                               |
                 free                                          scan
```

```
                    -------------------------------------------
                    |                                         |
                    v                                         |
     -------------------------------------------------------------------
     |     |     |     | X X | X X |     | X X |     |     |     |     |
     |  +  |  +  |  +  |  X  |  X  |  +  |  X  |  +  |  +  |  *  |     |
     |_____|_____|_____|_X_X_|_X_X_|_____|_X_X_|_____|_____|_____|_____|
                       ^                                   ^
                       |                                   |
                       |                                   |
                       free                              scan
```

```
     -------------------------------------------------------------------
     |     |     |     |     |     |     | X X |     |     |     |     |
     |  +  |  +  |  +  |  +  |  +  |  +  |  X  |  *  |  *  |  *  |     |
     |_____|_____|_____|_____|_____|_____|_X_X_|_____|_____|_____|_____|
                                   ^     ^
                                   |     |
                                   |     |
                                   free  scan
```

### Other algorithms

- Lisp 2 algorithm
- Threaded compaction
- One-pass algorithm

### Caveats

1. Object belongs to C-extension: the references in C cannot be updated, so the objects cannot be moved; therefore, they must be marked as pinned to avoid accidentally moving.
2. `object_id` is computed based on memory address of the object. After moving, the `object_id` shouldn't be changed. The solution is keeping a table to store the object ids of objects
3. Currently `GC.compact` doesn't do automatic GC yet. It also isn't very efficient because it does:
    - Full GC
    - Move objects
    - Update reference
    - Full GC

    The full GC is run twice because the mark and clean up steps aren't decoupled from GC yet.

## References

- Aaron Patterson PR for GC.compact: [https://bugs.ruby-lang.org/issues/15626](https://bugs.ruby-lang.org/issues/15626)
- Garbage collection's handbook
