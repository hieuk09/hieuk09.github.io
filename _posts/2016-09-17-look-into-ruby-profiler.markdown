---
layout: post
title: "Look into Ruby Profiler"
date:   2016-09-17
categories: [ruby]
---

# Look into Ruby Profiler

## Introduction

Ruby has a built-in profiler which is quite convenient for measuring performance
of a piece of code. For example, you can write:

```shell
ruby -rprofile example.rb
```

It will output a report after the execution is over:

```
%   cumulative   self              self     total
time   seconds   seconds    calls  ms/call  ms/call  name
68.42     0.13      0.13        2    65.00    95.00  Integer#times
...
```

## Look into Profiler code

### TracePoint Object

We'll look into module `Profiler` to see how it work.
However, before that, we'll talk about `TracePoint` first.

`TracePoint` is a class written in C, which is a hook that can listen to all
available events of ruby code: code execution, class/module definition, ruby
methods calls, c routine calls, exception and thread begin/end.

TracePoint provides some symbols to represent those events:

- :call - call a ruby method
- :return - end of a ruby method
- :c_call - call a C routine
- :c_return - end of a C routine
- :b_call - call a block
- :b_return - end of a block
- some other events

### How Profiler works

Now we go back to ruby profiler: [ruby code of module
Profiler](https://github.com/ruby/ruby/blob/32674b167bddc0d737c38f84722986b0f228b44b/lib/profiler.rb)
shows that it creates two procs called `PROFILE_CALL_PROC` and
`PROFILE_RETURN_PROC`. Each of them is a `TracePoint` object, which hooks to the
beginning and the end of the call to a piece of code.

As their names suggest, `PROFILE_CALL_PROC` captures all `call` events
and `PROFILE_RETURN_PROC` captures all `return` events. In addition to thoses,
Profiler also has an array of stacks to record all the events, with each stack
represents a thread. Lastly, it uses the system timer to calculate the execution
time.

When a piece of code starts, `PROFILE_CALL_PROC` will be called, and it'll
creates a log, which include a timer and an initialize value (start at 0).
When that piece end, `PROFILE_RETURN_PROC` will be called, it will get the log
that `PROFILE_CALL_PROC` creates and create another log (called map) which
stores:

- A key which is class name and method id
- Counter for the number of time that piece of code is called
- Total time the call run
- Total time the system spend in this call

## Conclusion

The basic idea of a profiler is extremely simple: record the beginning and the
end of a call, then substract to see the difference.
If looking deeper at the code (especially TracePoint class), there
probably is a lot more complexity behind all the magic. However, this could be a good
start to begin looking deeper into one of the most useful tool of
developer, a profiler.
