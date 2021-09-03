---
layout: post
title: "Fix incompatible libary version for gem with native extensions"
date:   2021-09-03
categories: ruby
---

Recently, I run into an issue when running `bundle exec`, then somehow
the command breaks with the below error:

```
An error occurred while loading spec_helper. - Did you mean?
                    rspec ./spec/spec_helper.rb

Failure/Error: require_relative '../config/environment'

LoadError:
  incompatible library version - /Users/hieunguyen/code/guardhouse/vendor/bundle/ruby/3.0.0/gems/byebug-11.1.3/lib/byebug/byebug.bundle
```


Searching through the internet, the issue happens because Clang enables
`-Werror=implicit-function-declaration` by default. We need to disable it when
building gem using `-Wno-error=implicit-function-declaration` option. For
example, to apply for `pg` gem:

```
bundle config build.pg --with-cflags="-Wno-error=implicit-function-declaration"
```
