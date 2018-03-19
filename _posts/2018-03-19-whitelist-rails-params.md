---
layout: post
title: "Whitelist Rails parameters with dry-validation"
date:   2018-03-19
categories: ruby
---

[dry-validation](http://dry-rb.org/gems/dry-validation/) is a powerful validation
library built by [solnic](https://github.com/solnic). I have been using it to replace
[strong-parameters](https://github.com/rails/strong_parameters) in my rails app,
but there are still some legacy piece of code uses strong-parameters
to whitelist params.

Oneday, I decide that I don't want to switch my brain from thinking in dry-v to
strong-parameters anymore. Therefore, I convert all the legacy code to use
dry-validation.

So, I have a piece of code like this in `users_controller.rb`:

```ruby
params.required(:user).permit(:password, :password_confirmation)
```

To make sure not to change the functionality of my old code, I write a few
tests:

```ruby
describe UserSchema do
  describe '#call' do
    let(:result) { described_class.call(input) }

    context 'when user is missing' do
      it 'returns error' do
        # some code here
      end
    end

    context 'when user is present with data' do
      it 'validates successfully' do
        # some code here
      end
    end
  end
end
```

Then I implement the dry-validation schema to make the tests pass:

```ruby
UserSchema = Dry::Validation.Schema do
  required(:user).schema do
    optional(:password)
    optional(:password_confirmation)
  end
end
```

However, this is not all functionality that strong-parameters provide, it also
filters all parameters that are not whitelist. I add a test:

```ruby
context 'when user data has malicious params' do
  let(:input) { { user: { admin: true } } }

  it 'returns only whitelist data' do
    expect(result).to be_success
    expect(result.output).to eq(user: {})
  end
end
```

Then this test fails. It allows `admin` in the output hash. `Schema` sanitizer
only checks the specified parameters, it's ignored other parameters unless we
configured it to use `sanitizer`. However, there are built-in `Form` options
that filters all those unwanted value. I change the schema to:

```ruby
UserSchema = Dry::Validation.Form do
  required(:user).schema do
    optional(:password)
    optional(:password_confirmation)
  end
end
```

The test passes, but the previous test that checks `password` and
`password_confirmation` fails. It's because the sanitizer filters all the value
that does not specified type. I'm not sure if it's a bug or a feature here. I
update the schema to:

```ruby
UserSchema = Dry::Validation.Form do
  required(:user).schema do
    optional(:password).maybe(:str?)
    optional(:password_confirmation).maybe(:str?)
  end
end
```

Now all the tests pass. I think that although the schema is more verbose than
the code using strong-parameters, it offers more flexibility and information.
