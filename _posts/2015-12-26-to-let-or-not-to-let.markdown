---
layout: post
title:  "To let or not to let?"
date:   2015-12-26
categories: [testing]
---

This is the English version of [my original post][original-kipalog-post].

## `Let` gives us a lot

When we (Ruby developers) write test, be it in rspec or minitest, we usually
use `let`. `Let` method helps us write code easier and more convenient.

{% highlight ruby %}
def activable?
  inactive? && !blacklist?
end

describe '#activable?' do
  let(:inactive?) { true }
  let(:blacklist?) { false }
  subject { build_stubbed(:user) }

  before do
    expect(subject).to receive(:inactive?).and_return(inactive?)
    expect(subject).to receive(:blacklist?).and_return(blacklist?)
  end

  context 'when user is active' do
    let(:inactive?) { false }
    it { is_expected.not_to be_activable }
  end

  context 'when user is blacklist' do
    let(:blacklist?) { true }
    it { is_expected.not_to be_activable }
  end

  context 'otherwise' do
    it { is_expected.to be_activable }
  end
end
{% endhighlight %}

As you can see, in each context, we override a single value correspond to that
context. This kind of syntax gives us two advantages:

- Avoid duplication. We can write shorter and more readable code.

- Easily reveal the conditions of each context. Reader will know exactly the
  condition when a user is not activable.

However, in a recent conversation with a colleague, I found out that some
companies like Thoughbot or Thoughworks had forbid their developers from using
this method.

## Problem with `let`

There is a talk from Thoughtbot engineers about ["Why we shouldn't use let"]
[thoughtbot-video], it basically talked about two
problems:

- The setup phase is easily mixed with execution or assertion phase due to lazyload
  nature of `let`. That can become a hidden bug when your tests grow or you make
  a mistake. You can see in the example below, do you know why the test failed?

{% highlight ruby %}
describe '.active' do
  let(:user) { create(:user, active: active) }
  subject { User.active }

  describe 'when user is active' do
    let(:active) { true }
    it { is_expected.to eq user } # this test will fail
  end

  # ...
end
{% endhighlight %}

- The test setup are highly couple together, and you can make the whole test
  suite failed with a single change in the setup data. For example, you can take
  a look at [this piece of
  code][example-with-let].

In this example, a junior (or even a senior) will probably have hard time find
out the cause of the error in case the test fails because some variables (like
`attributes`) are override too many times. They can't separate the test setup and
the test execution either.

## Solution

In the talk, the Thoughtbot engineers propose some rules to solve the above
problems when writing tests:

- Allow code duplication

- Write test in 4-phases: setup, execution, assertion and teardown should be
  separate clearly

- Don't use `let`, `subject` or similar method

I refactored the above tests with these rules, and this is the
[result][example-without-let].

As you can see, this piece of code is easier to read and debug to the original
piece. We know exactly how the data is prepared, and those data is setup
separately, so we can figure out how to change them without breaking other tests
effortlessly.

## Limitation

After trying this for a while, even with the above advantages, the solution
without less still has some limitations:

- When writing longer tests, the duplication become a hindrance for writing
  clean code.

- When I refactor them, I found myself go back to the beginning (like with using
  `let`) because I've just added a layer of redirection and abstraction. My
tests become couple with the abstraction. Therefore, when I change something, I
face the risk of breaking something else again.
However, in contrast with `let`, my abstraction is in my code, so I can freely manipulate it,
and I can apply some OPP or FP pattern to reduce the risk. I'm still thinking of
another better solution, though.

## Summary

`let` method is a very convenient way for you to write tests with not so much
duplication. However, it will couple your tests with the data setup, and make
tests hard to change. You may remove it to obtain the separation of concern, but
will have to face the duplication, and it seems to be a reasonable price to
paid.

[original-kipalog-post]: http://kipalog.com/posts/DUNG-LET-HAY-KHONG
[thoughtbot-video]: https://upcase.com/videos/rspec-best-practices
[example-with-let]: https://gist.github.com/hieuk09/a31e8f8c5fcdd2d7e9bf#file-acceptance_use_let_spec-rb
[example-without-let]: https://gist.github.com/hieuk09/a31e8f8c5fcdd2d7e9bf#file-acceptance_no_let_spec-rb
