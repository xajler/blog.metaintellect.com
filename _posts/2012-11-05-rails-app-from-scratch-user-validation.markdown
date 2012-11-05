---
layout: post
title: "Rails App from Scratch: User Validation"
date: 2012-11-05 13:30
comments: true
categories: [rails, ruby, TDD, rspec, validation]
keywords: rails, ruby, TDD, rspec, validation
description: The Rails 3 App from Scratch, implementing a User Validation.
---

On the last post we've created a User Sign Up page and in this post we'll continue to add
some validation to the User to make sure that User is valid so it is safe to save or update
User to the Database.

## Validation

There are few rules that we've mention in last post about some validation logic:

* _Email_ - Required, Unique, valid email format and after creation it cannot be updated anymore or for short read-only.
* _Password_ - Required and at least 8 characters long.
* _Full Name_ - Required.

## Validation for Required Attributes

We had one pending test up till now in `spec/models/user_spec.rb`. We will use this test
file to set up the Validation for User in TDD way.

First remove pending part generated on User Model:

    pending "add some examples to (or delete) #{__FILE__}"

### User Factory

First lets create a Factory of User that we'll use in tests. This will be a valid Model of User:

```ruby
FactoryGirl.define do
  factory :user do
    email 'xajler@gmail.com'
    password 'x1234567'
    password_confirmation 'x1234567'
    full_name 'Kornelije Sajler'
  end
end
```
### RSpec let and subject and Factory Girl build

Then we'll learn some of RSpec and Factory Girl, but first add this beneath the `describe` block:

```ruby
let :user do
  build :user
end

subject do
 user
end
```

RSpec `let` description borrowed from the RSpec documentation:

> Use let to define a memoized helper method. The value will be cached across multiple calls in the same example but not across examples.

> Note that let is lazy-evaluated: it is not evaluated until the first time the method it defines is invoked. You can use let! to force the method's invocation before each example.

RSpec `subject`

> Use subject in the group scope to explicitly define the value that is returned by the subject method in the example scope.

There is also used a Factory Girl `build` that returns a User instance that's not saved, use `create`
if it is mandatory that Model is saved to database before getting it in tests.

### Required Fields Model Tests

The `email`, `password` and `full_name` are required so we create the RSpec `context` named
_is invalid_ and even though we should go one by one test for each attribute, for quickness we'll
do them at once:

```ruby
context 'is invalid' do
  it 'when required #email is not given' do
    user.email = ''
    should_not be_valid
  end

  it 'when required #password is not given' do
    user.password = ''
    should_not be_valid
  end

  it 'when required #full_name is not given' do
    user.full_name = ''
    should_not be_valid
  end
end
```
> Note:
>
> The `#` is used to denote the Ruby way of describing the _instance methods_, the `.` is
> used for the _class methods_!

The `should_not` can be used since we set a `subject` to be instance of User built from
Factory Girl `:user` factory so the RSpec knows to what the `should_not` refers to.

The `be_valid` method is a RSpec shorthand for the Rails `valid?` method that returns
boolean hence the `?`, every Ruby method with `?` can be called in RSpec with `be_<name_of_method>`.

The running tests should failing with message:

```
Failures:

  1) User is invalid when required #full_name is not given
     Failure/Error: should_not be_valid
       expected valid? to return false, got true

  2) User is invalid when required #email is not given
     Failure/Error: should_not be_valid
       expected valid? to return false, got true

  3) User is invalid when required #password is not given
     Failure/Error: should_not be_valid
       expected valid? to return false, got true
```

To make the test green, add the `validates presence` for all three required properties
in the `app/models/user.rb`:

```ruby app/models/user.rb
class User < ActiveRecord::Base
  has_secure_password

  attr_accessible :email, :password, :password_confirmation, :full_name

  validates :email, presence: true
  validates :password, presence: true
  validates :full_name, presence: true
end
```

The tests should all pass:

    Finished in 0.81772 seconds
    5 examples, 0 failures

## Validation of Email Uniqueness

Another validation for `email` is that is need to be unique or there should not be two same
`email`s stored in the database.

Add the new `it` test to `spec/models/user_spec.rb` in _is invalid_ `context`:

```ruby
it 'when #email is not unique' do
  user.save
  user1 = build :user
  user1.save

  user1.should_not be_valid
  user1.errors.full_messages[0].should match 'Email has already been taken'
end
```

This test is little sketchy, firstly because there are two assertions and secondly of
saving our `subject` User, then `build` identical User, store him to `user1` variable, and
then try to save User to database.

The second assertion is just to make sure that error is raised because of the `email` uniqueness.

The failing message:

```
1) User is invalid when #email is not unique
   Failure/Error: user1.should_not be_valid
     expected valid? to return false, got true
```

So the only thing is for us to prevent having `email` stored to database more than once
with `uniqueness` added to existing `email vaildates`:

```ruby
validates :email, presence: true, uniqueness: true
```
This should make the test green:

    Finished in 0.8653 seconds
    6 examples, 0 failures

## Validation of Email format

Next there is need to make sure that the `email` format is valid. The _Regular Expression_
is used to validate the `email` format.

> **Note**: There are better ways to do the complex Mail validation in Ruby or Rails, but it is out of scope of this simple app!

Add test below latest one, still in the _is invalid_ `context`:

```ruby
it 'when #email format is not valid' do
  user.email = 'invalid mail'
  should_not be_valid
end
```
The test should fail with message:

```
1) User is invalid when #email format is not valid
   Failure/Error: should_not be_valid
     expected valid? to return false, got true
```

To fix it simple as possible add the `format` to `email validates`:

```ruby
validates :email, presence: true, uniqueness: true,
          format: { with: /\A[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]+\z/,
                    message: 'The format of Email is invalid'}
```

This should make to pass the test:

```
Finished in 1.3 seconds
7 examples, 0 failures
```

## Validation of at least 8 chars for Password

The User entered `password` must be at least 8 characters.
The `password` will also be simple as possible without checking that there are at least
one number or symbol, but rather, just to have at least 8 characters!

Add new test as last in `context` _is invalid_:

```ruby
it 'when #password is not at least 8 characters' do
  user.password = 'abc123'
  should_not be_valid
end
```

The test should fail with message:

```
1) User is invalid when #password is not at least 8 characters
   Failure/Error: should_not be_valid
     expected valid? to return false, got true
```

To make test pass add the `length` to the `password validates`:

```ruby
validates :password, presence: true, length: { minimum: 8 }
```

The test should pass now:

    Finished in 1.09 seconds
    8 examples, 0 failures

## The Email must be read-only

The `email` can only be set when is created and after saving to the database that `email`
must not ever be possible to change.

Outside of the `context` _is invalid_ create the new `it` test:

```ruby
it '#email must not ever change after it is created' do
  user.save
  user.update_attributes email: 'ksajler@gmail.com'
  user.reload.email.should eql 'xajler@gmail.com'
end
```

This test is a little bit weird, what it ensures that when attributes are updated and
reloaded from the database, that the `email` is still same as when it was created even
though is changed to new value.

The test should fail with message:

```
1) User#email must not ever change after it is created
   Failure/Error: user.reload.email.should match 'xajler@gmail.com'
     expected "ksajler@gmail.com" to match "xajler@gmail.com"
```

To make sure that `email` is never changed after creation and all attempts to change the
`email` will be silently ignored, use Rails `attr_readonly` for `email`.

Then the `app/models/user.rb` should look like this:

```ruby app/models/user
class User < ActiveRecord::Base
  has_secure_password

  attr_accessible :email, :password, :password_confirmation, :full_name
  attr_readonly :email

  validates :email, presence: true, uniqueness: true,
            format: { with: /\A[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]+\z/,
                      message: 'The format of Email is invalid'}
  validates :password, presence: true, length: { minimum: 8 }
  validates :full_name, presence: true
end
```

And the test should pass:

    Finished in 1.28 seconds
    9 examples, 0 failures


## Test that User is valid

We tested all invalid combinations of the User to make sure that the logic we wanted is
implemented, now for sanity check we'll add the test to make sure when all given is valid
then the User should be valid and saving of the User can be executed.

Add new test `is valid` and the whole `spec/models/user_spec.rb` should look like this:

```ruby spec/models/user_spec.rb
require 'spec_helper'

describe User do
  let :user do
    build :user
  end

  subject do
   user
  end

  context 'is invalid' do
    it 'when required #email is not given' do
      user.email = ''
      should_not be_valid
    end

    it 'when required #password is not given' do
      user.password = ''
      should_not be_valid
    end

    it 'when required #full_name is not given' do
      user.full_name = ''
      should_not be_valid
    end

    it 'when #email is not unique' do
      user.save
      user1 = build :user
      user1.save

      user1.should_not be_valid
      user1.errors.full_messages[0].should match 'Email has already been taken'
    end

    it 'when #email format is not valid' do
      user.email = 'invalid mail'
      should_not be_valid
    end

    it 'when #password is not at least 8 characters' do
      user.password = 'abc123'
      should_not be_valid
    end
  end

  it '#email must not ever change after it is created' do
    user.save
    user.update_attributes email: 'ksajler@gmail.com'
    user.reload.email.should match 'xajler@gmail.com'
  end

  it 'is valid' do
    should be_valid
  end
end
```

And all 10 tests, Integration and Model should pass:

    Finished in 1.24 seconds
    10 examples, 0 failures

## Prettier RSpec Tests

By default the RSpec tests are represented as dots (`.`) if they are passed and `F` if they fail.

To display `describe`, `context` and `it` titles while running RSpec, add `format` to
`.rspec` file:

```text .rspec
--color
--format documentation
```

Run all test and the format of RSpec test should look like this:

```text
User
  #email must not ever change after it is created
  is valid
  is invalid
    when #password is not at least 8 characters
    when required #password is not given
    when #email format is not valid
    when required #full_name is not given
    when #email is not unique
    when required #email is not given

Users
  GET /users/new
    displays the create new user page
  GET /signup
    displays the sign up page

Finished in 1.81 seconds
10 examples, 0 failures
```

## Conclusion

This post was all about the Rails Validation, there are few interesting samples how to
test Rails application. The TDD in this post is solely done on User Model instead of on
request browser based testes done with _Capybara_.

Now when we are sure that User Model validation logic is implemented and tested in next
post, we will Save and Update User Views, Controller methods and create browser based
test to make sure that User Model logic actually works in real usage!

## Code

The code is hosted on GitHub and can be cloned from the [xajler/just-todo-it](https://github.com/xajler/just-todo-it).

> Github xajler/just-todo-it commit for this post:
>
> [Implemented User Model validation and tested in user_spec.rb, the tests using users factory.](https://github.com/xajler/just-todo-it/commit/f2edd5551c915bba59b3aabb8c894325cf4f5414)
