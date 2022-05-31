---
layout: post
title: "Some Ruby tidbits"
date:   2022-05-31
categories: ruby
---

Today I watched [A Day in the Life of a Ruby Object - Jemma Issroff](https://www.youtube.com/watch?v=PuNbdfdFBjk),
which is a very educational video about how Ruby handles memory. It also touches
a bit on how GC works in ruby, too.

I learnt from this video:

- If an object needs more memory than a slot in Ruby heap, Ruby allocates memory
inside OS heap instead. Therefore, it's possible that our program uses more
memory than what is shown as the memory consumption of a Ruby process.

- The ruby slot size is 16kb, and it can store something like a 23-characters
string. If we allocate a bigger object, like 24-characters string, Ruby
allocates memory inside OS heap. There is a performance hit in that case.

- We can use `ObjectSpace.dump` to dump memory information of an object (we need
    to require `objspace` before using).

- We can use `GC.stat` to show GC information
