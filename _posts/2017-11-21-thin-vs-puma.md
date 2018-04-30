---
layout: post
title: "Calculate total page load using cucumber"
date:   2017-11-21
categories: ruby
---

A few days ago, when doing code review for a colleguage, I have a discussion
with him about using whether we should remove `thin` from our codebase and
replace it with `puma` instead. His reasons are:

- Because our production server also uses puma, we'll have more dev prod parity that way
- `puma` is faster than `thin`
- Rails 5 makes puma default for development. One fewer dependency to worry out.

The first reason is totally valid, and the last point is not really relevant (we
still use rails 4), so we discuss about the second point here.

To be honest, I like `puma`, and I agree that it's better than most webservers
for Rails app in production. However, running server in local have different
problems, and a single thread server is usually ideal for development
(especially in ruby).

On another hand, I don't like having confirmation-bias, and while `puma` is
believe to be faster than `thin`, I would like to see how they are compared to
each other in our specific situation.

## TINYpulse cases:

TINYpulse is a website consisted of several parts: some pages are
single-page-application (SPA) while some pages are still rails views. This leads
to different styles of request serving: SPA relies on a lot of API calls, while
rails views is rendered only once from the controller, then is served to the
users browser.

In order to correctly assert how fast the page is load to a developer, I check
whether or not all necessary elements are rendered and visible. Capybara
provides all the features to assert element presence and visibility, so I use it
to run the benchmark.

## Setup server:

Capybara provides a simple way to customize the server to run your rails application:

```ruby
Capybara.register_server :puma do |app, port, host|
  require 'rack/handler/puma'
  Rack::Handler::Puma.run(app, Port: port, Host: host)
end

Capybara.register_server :thin do |app, port, host|
  require 'rack/handler/thin'
  Rack::Handler::Thin.run(app, Port: port, Host: host)
end

Capybara.server = :thin
```

Through `tag`, we can customize different server running for each tests, so that
we don't have to manually change the code and rerun the test:

```ruby
# This configuration is for cucumber, but you may do the similar thing for rspec as well

Before('@thin') do
  Capybara.server = :thin
end

Before('@puma') do
  Capybara.server = :puma
end
```

## Benchmark:

In the test, we can have a block of code to calculate the time to load a page:

```ruby
Benchmark.bm do |x|
  x.report do
    visit dashboard_path
    assert_dashboard_visible
  end
end
```

With this, the test will print the output of the page load time to console log.
We can also create a loop to load the page multiple times so that the page load
time is more accurate. We can now use the benchmark to assert the initial
problem:

### SPA page load:

|                     | Thin   | Puma (1 worker - 1 thread) | Puma (2 worker - 5 thread) |
|---------------------|--------|----------------------------|----------------------------|
| Time taken          | 97.707 | _106.183_                  | *62.638*                   |
| Completed requests  | 20     | 20                         | 20                         |
| Failed requests     | 0      | 0                          | 0                          |
| Time per requests   | 4.885  | _5.309_                    | *3.132*                    |
| Requests per second | 0.205  | _0.188_                    | *0.319*                    |

### Rails view page load:

|                     | Thin   | Puma (1 worker - 1 thread) | Puma (2 worker - 5 thread) |
|---------------------|--------|----------------------------|----------------------------|
| Time taken          | 46.470 | _79.226_                   | *35.842*                   |
| Completed requests  | 20     | 20                         | 20                         |
| Failed requests     | 0      | 0                          | 0                          |
| Time per requests   | 2.323  | _3.961_                    | *1.792*                    |
| Requests per second | 0.430  | _0.252_                    | *0.558*                    |

## Conclusion

As we can see from the benchmark, Puma with multiple workers and multiple
threads is superior in term of speed to thin for both SPA and rails view page;
however, when we needs to set it to single thread mode for debugging purpose,
speed will be downgraded to lower than thin.

In development workflow, I believe that we spend more time to debugging and
reading the console log than checking the page view (except for front-end devs),
so I think thin is still a more viable option for us in development than puma.
