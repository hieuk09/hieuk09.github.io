---
layout: post
title: "ActiveRecord 3.2 bug: Can not save when mutate string attribute"
date:   2016-01-17
categories: activerecord
---

This is the English version of [my original post][original-kipalog-post].

*Note*: This bug is fixed from ActiveRecord 4.0

## The weird bug

{% highlight ruby %}
user = User.first # => #<User id: 1, email: "hieu@tinyhr.com">
user.email.gsub!('tinyhr', 'tinypulse')
user.save

User.first.email # => still "hieu@tinyhr.com"
{% endhighlight %}

## The cause

When you use `save`, Rails check to see whether the attributes are changed
before update the record in database.  To optimize that process, it used
the reference instead of the value of the attributes. Therefore, when you
mutate the attribute (using `gsub!` or something similar), the reference
won't change, and it won't pass the check.

## How to fix

Simply create another reference for the attribute:

{% highlight ruby %}
user = User.first # => #<User id: 1, email: "hieu@tinyhr.com">
user.email = user.email.gsub('tinyhr', 'tinypulse')
user.save

User.first.email # => "hieu@tinypulse.com"
{% endhighlight %}

## Lession learn

Mutation lead to a lot of side-effect bugs like this, so I try to avoid it
whenever possible. It's just so convenient in Rails, though...

[original-kipalog-post]: http://kipalog.com/posts/Workaround-cho-loi-khong-luu-duoc-mutate-string-trong-activerecord
