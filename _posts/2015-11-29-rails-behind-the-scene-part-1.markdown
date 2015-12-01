---
layout: post
title:  "Rails, behind the scene (part 1): finder methods
subtitle: "One of the most convenient things of Rails is its `find` method"
date:   2015-11-29 23:34:01
categories: [rails, monkey-patching]
---

# Introduction

- 2 types of developers in RoR world: Ruby developers & Rails developers
- Rails is just ruby
- This series of articles are for people to understand the internal architechture
  and technique of rails => more effective at work

# Monkey-patching

- A technique to modify the existing class without having to create a new class
- [A very good and simple slide](https://speakerdeck.com/hqc/monkey-patching-in-ruby)

# How it is used?

- The magic of Rails come from monkey-patching 2 methods:
  + `const_missing`
  + `method_missing`

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}
