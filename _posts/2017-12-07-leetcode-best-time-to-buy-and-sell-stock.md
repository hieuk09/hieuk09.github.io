---
layout: post
title: "Leetcode - Best time to buy and sell stock"
date:   2017-11-29
categories: programming algorithm
---

Recently, our chat group in [RubyVN](chat.ruby.org.vn) starts a channel called `#algorithm` for discussing competitive programming.
This week, this is one of the three problems that we will discuss:

# Problem:

[https://leetcode.com/problems/best-time-to-buy-and-sell-stock/](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)

Level: easy

# Solution

## Naive approach

Loop through all possible combinations to find the combination that has max
profit:

```ruby
def max_profit(prices)
  size = prices.size
  max = 0

  prices.each_with_index do |price, index|
    prices[(index+1)..(size-1)].each do |price_2|
      max = [max, price_2 - price_1].max
    end
  end

  max
end
```

This solution needs to one nested loop to get all combinations, so its time
complexity is O(n^2), which is not so good. It will cause Time Limit Exceed on
leetcode.

## First optimization

With observation, we can figure out the equation for max profit of a
position i:

```ruby
max_profit[i] = max[i, size - 1] - min[0, i]
```

We can cache `max[i, size - 1]` and `min[0, i]` to two arrays, then calculate
the max profit based on those:

```ruby
def max_profit(prices)
  min = 0
  min_array = prices.map do |price|
    min = [min, price].min
  end

  max = 0
  max_array = prices.reverse.map do |price|
    max = [max, price].max
  end

  (0..(prices.size - 1)).inject(0) do |profit, index|
    [profit, max_array[index] - min_array[index]].max
  end
end
```

This solution needs 3 loops through the array, so it's 3xn, or O(n) complexity.
It's also need 2 more arrays to cache the value of `max[i, size-1]` and `min[0, i]`.

## Final optimization

With further observation, we can see that we don't need two arrays to cache the
value. Instead, we can calculate them on the fly:

```ruby
def max_profit(prices)
  return 0 if prices.empty?

  min = prices.first
  profit = 0

  prices.each do |price|
    min = [price, min].min
    profit = [price - min, profit].max
  end

  profit
end
```

This solution requires only 2 temp variables and only one loop, so it's a little
bit better than the previous optimization.
