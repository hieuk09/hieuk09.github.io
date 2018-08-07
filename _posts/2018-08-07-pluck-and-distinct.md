---
layout: post
title: "TIL - Wierd behaviors of pluck and uniq in ActiveRecord"
date:   2018-08-07
categories: activerecord
---

Normally, when fetching raw columns data from database in ActiveRecord,
  `pluck` is prefered to `map` because it only fetches the necessary columns and
  does not create any ActiveRecord objects:

```ruby
# fetch all columns and create activerecord objects
Product.all.map(&:name)

# fetch only necessary columns and create activerecord objects
Product.select(:name).map(&:name)

# fetch only necessary columns and create only necessary primitive objects
Product.pluck(:name)
```

However, when using with `uniq` and non-top-level scope, it can catch us offguard:

```ruby
Product.uniq.pluck(:name) # works, only unique names are returned

category.products.uniq.pluck(:name) # does not work, all names are returned
```

In that case, we can use `uniq` after the `pluck`, but it may cost us a lot of
memory to handle all the returned data:

```ruby
category.products.pluck(:name).uniq # works, but waste a lot of memory to store redundant data
```

A faster solution is using `DISTINCT` keyword directly in `pluck`, but it may
harm readability/maintainability:

```ruby
category.products.pluck('DISTICT name') # works, but may fail if name is an aliased column
```

I wonder why there is this wierd behavior...
