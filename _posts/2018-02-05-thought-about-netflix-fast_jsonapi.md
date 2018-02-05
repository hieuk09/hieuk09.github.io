---
layout: post
title: "Thought about Nefflix's fast_jsonapi gem"
date:   2018-02-05
categories: ruby
---

A few days ago, I came across [a tweet of Aaron
Peterson](https://twitter.com/tenderlove/status/959924273811894275) that
introduces a new gem made by Netflix:
[fast_jsonapi](https://github.com/Netflix/fast_jsonapi). I've worked with a few
json rendering library over the years, and haven't found anything that 100% satisfy
my need, so I'm earger to take a look.

## Pros

After a trying it on one of my pet project to replace AMS (which it claims to
replace), I found that it is really simple to finish the task with the gem. If you have an AMS
serializer:

```ruby
Class Books::Author < ActiveModel::Serializer
  attributes :name, :birthday, :info
end
```

You just need to remove `ActiveModel::Serializer` inheritance, and include `FastJsonapi::ObjectSerializer`
instead. Now you have:

```ruby
Class Books::Author
  include FastJsonapi::ObjectSerializer
  attributes :name, :birthday, :info
end
```

You also need to replace `as_json` by `serialized_hash` and `to_json` by
`serialized_json`, though.

I looked at the dependencies of the gem, and see that it uses `oj`,
which is nice. No wonder it has such good speed!

However, the more I work with it, the more I feel there are something missing:

## Cons

It requires Rails (Rails 5 in particular, though there are support for
Rails 4.9), so if you don't use Rails, you cannot use it without include the
whole Rails dependencies. If you use Rails less than 5 (or 4.9), you cannot use
it without upgrading to Rails 5.

It supports `oj` natively also mean that you cannot use any other alternative of
json rendering library, like `json` or [yail](https://github.com/brianmario/yajl-ruby).
If you prefer those library for some reason, you cannot use this gem as well.

Another quirk that find found a little uncomfortable with is that this gem treats
the input differently based on its type. If the input is `Enumerable` (like
`Array` or `ActiveRecord::Relation`), it works as collection serializer.
Otherwise it work as object serializer. While it is not really that bad, there
will be usecase that you don't really need this feature.

I also miss a few features from other json libraries like custom element name,
or specify render or not render nil object, and a few other features. Of course,
those will come with time, as there are a lot of PR that is submitting to it
recently.

The last one is: what is claim is not really the truth. I see the benchmark that
it's multiple time faster than AMS and suprise. I had
[similar benchmark](https://github.com/hieuk09/benchmark_json_renderer) a while
ago, and while AMS is not the fastest, when it comes to render a complex
dataset, it's not really that bad. Therefore, I run my benchmark again with
`fast_jsonapi` and see the result below:


### Benchmark pure rendering

![pure_rendering_collection]({{ site.url }}/assets/pure_rendering_collection.png)


### Benchmark rendering through API call

![api_rendering_collection]({{ site.url }}/assets/api_rendering_collection.png)

As you can see: for simple collection rendering, it's quite fast, totally
beating AMS and almost all other gems. However, for the most complex case, it
fails miserably compared to `AMS`, `Roar` and `Grape::Entity`

## Summary

To sum it up, while `fast_jsonapi` is a nice gem, it doesn't add much to what we
currently have, in term of usability or performance. Other mature gems such as
`Grape::Entity` or `Roar`, with a lot more features and more consistent speed,
may be a better choice for your application.

## Update

[According to the author of the gem](https://github.com/Netflix/fast_jsonapi/issues/11),
the gem is focusing only on optimizing rendering json in jsonapi spec,
that's why it's much more faster than AMS with json api adapter
(which is significant slower than default adapter).
