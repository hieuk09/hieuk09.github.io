---
layout: post
title: "Operators precendence in Ruby"
date:   2018-05-12
categories: ruby
---

Previously, I thought that `&&` and `||` have the same precendence in Ruby;
however, today I know that `&&` has higher precendence than `||`, so `&&` is
always evaludated before `||`. I look more into the
[precedence reference](https://ruby-doc.org/core-2.2.0/doc/syntax/precedence_rdoc.html)
and find some interesting notes:

- Greater/Less operators has higer precendence than equal operators
- `&&` has higher precendence than `||`, but `or` has the same precendence as
`and`
- `&` also has higher precendence than `|`
