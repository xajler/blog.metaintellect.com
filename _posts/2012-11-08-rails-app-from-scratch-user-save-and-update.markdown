---
layout: post
title: "Rails App from Scratch: User Save and Update"
date: 2012-11-08 01:25
comments: true
categories: [rails, ruby, TDD, rspec, authentication]
keywords: rails, ruby, TDD, rspec, save, update, authentication
description: The Rails 3 App from Scratch, implementing a User Save and Update.
---

After we implemented logic around User attributes, it is time to finally save and update
the User to the database.

But first, we shall change a mistake from the post on creating Sign Up page, the `password`
fields in the View are set as normal text. We shall make sure that it is a password field!

Change the `text_field` to `password_field` in `app/views/users/_form.html.haml`:

```haml
= f.label :password, 'Password:'
= f.password_field :password
%br

= f.label :password_confirmation, 'Confirm Password:'
= f.password_field :password_confirmation
%br
```

## Saving Valid User Test

Now when the password fields are fixed, lets create a test to save a Valid User to the
database.

Create a new `context` for `POST` in `spec/requests/users_spec.rb`:

```ruby
context 'POST /users' do
  it 'creates and saves the valid user' do
    visit new_user_path

    fill_in 'Email', with: 'xajler@gmail.com'
    fill_in 'Password', with: 'x1234567'
    fill_in 'Confirm Password', with: 'x1234567'
    fill_in 'Full Name', with: 'Kornelije Sajler'
    click_button 'Sign Up'

    current_path.should == signup_path
    page.should have_content 'The User is successfully saved!'
  end
end
```

The test is pretty simple, we fill up the form with data, click 'Sign Up' button, and
expect to be redirected again to "Sign Up" page (in new post when we create a Login page
the `create` action will be redirecting to it!) and we expect that the flash notice will show that
User is successfully saved.

The test should fail on the clicking the _Sign Up_ button, because there is no `create`
action in `UsersController`:

```
1) Users POST /users creates and saves the valid user
   Failure/Error: click_button 'Sign Up'
   AbstractController::ActionNotFound:
     The action 'create' could not be found for UsersController
```

So to pass this test we need to add a `create` method to the `UsersController`:

```ruby
def create
  @user = User.new params[:user]

  if @user.save
    flash[:notice] = 'The User is successfully saved!'
    redirect_to signup_path
  end
end
```

First we storing a User to instance variable `@user` from the data given in User Sign Up
Form as `params[:user]`. Then we try to save it, and on success, we assign the flash notice
and redirect back to Sign Up Page.

The test should pass:

    Finished in 1.37 seconds
    11 examples, 0 failures

The happy path is working, now lets test all the possible ways the Sign Up might go wrong!

## Invalid User Tests

### Password Confirmation

Lets first check `password`s matching is working, by applying the two different passwords.
We shall crate the new `context` inside `POST /users` `context` and call it "_not saving invalid user_":

```ruby
context 'not saving invalid user' do
  it 'when passwords mismatch' do
    visit new_user_path

    fill_in 'Email', with: 'xajler@gmail.com'
    fill_in 'Password', with: 'x1234567'
    fill_in 'Confirm Password', with: 'x123'
    fill_in 'Full Name', with: 'Kornelije Sajler'
    click_button 'Sign Up'

    current_path.should == signup_path
    page.should have_content "Password doesn't match confirmation"
  end
end
```

After running test we shall get odd message that there is no View Template for `create`.

```
1) Users POST /users not saving invalid user when passwords mismatch
   Failure/Error: click_button 'Sign Up'
   ActionView::MissingTemplate:
     Missing template users/create, application/create with {:locale=>[:en], :formats=>[:html], :handlers=>[:erb, :builder, :coffee, :haml]}. Searched in:
       * "/Users/xajler/src/rb/just-todo-it/app/views"
```

We don't need a new View Template but rather change the `UserController` method `create`
to do something if the user is not saved. And when is not saved we shall flash an error message:

```ruby
def create
  @user = User.new params[:user]

  if @user.save
    flash[:notice] = 'The User is successfully saved!'
    redirect_to signup_path
  else
    flash[:error] = @user.errors.full_messages[0]
    redirect_to signup_path
  end
end
```

We are setting as a flash message from a User instance errors and getting first `full_message`
that will be set upon having problem saving user and in this case message will be
"_Password doesn't match confirmation_".

After applying the given code, the test should pass:

    Finished in 1.54 seconds
    12 examples, 0 failures

With this code applied we also satisfy the tests that are yet to come, but we still need
tests to prove that `create` method is working and proper error messages are thrown to
guide the users of Application to resolve them!

### Required Fields

We have three required fields:

* `email`
* `password`
* `full_name`

Lets write three more test, they should all be green since the implementation is written
to satisfy the previous test.

We will add them to the "_not saving invalid user_" `context` below the latest test:

```ruby
it 'when email is blank' do
  visit new_user_path

  fill_in 'Email', with: ''
  fill_in 'Password', with: 'x1234567'
  fill_in 'Confirm Password', with: 'x1234567'
  fill_in 'Full Name', with: 'Kornelije Sajler'
  click_button 'Sign Up'

  current_path.should == signup_path
  page.should have_content "Email can't be blank"
end

it 'when password is blank' do
  visit new_user_path

  fill_in 'Email', with: 'xajler@gmail.com'
  fill_in 'Password', with: ''
  fill_in 'Confirm Password', with: ''
  fill_in 'Full Name', with: 'Kornelije Sajler'
  click_button 'Sign Up'

  current_path.should == signup_path
  page.should have_content "Password digest can't be blank"
end

it 'when full name is blank' do
  visit new_user_path

  fill_in 'Email', with: 'xajler@gmail.com'
  fill_in 'Password', with: 'x1234567'
  fill_in 'Confirm Password', with: 'x1234567'
  fill_in 'Full Name', with: ''
  click_button 'Sign Up'

  current_path.should == signup_path
  page.should have_content "Full name can't be blank"
end
```

So, in first test we test required Email, by sending blank one. In second the Passwords
are blank and in third the Full Name is blank. And each test expect that meaningful messages
are shown to the user of application!

And if all was OK then all tests should pass:

    Finished in 1.79 seconds
    15 examples, 0 failures

### Email Uniqueness

The saving User with entered existing `email` should fail. Lets create the test to prove
it:

```ruby
it 'when email is not unique' do
  create :user
  visit new_user_path

  fill_in 'Email', with: 'xajler@gmail.com'
  fill_in 'Password', with: 'x1234567'
  fill_in 'Confirm Password', with: 'x1234567'
  fill_in 'Full Name', with: 'Kornelije Sajler'
  click_button 'Sign Up'

  current_path.should == signup_path
  page.should have_content 'Email has already been taken'
end
```

So, first we insert User from our created factory via _Factory Girl_. The factory of User
inserted has the same email that we try to save it from the User Sign Up Form. And again
we test to get meaningful message when email is not unique.

The test should pass, because the code is implemented:

    Finished in 2.02 seconds
    16 examples, 0 failures

### Password with at least 8 characters

The User cannot be saved if both passwords length is at least 8 characters long:

```ruby
it 'when password is less than 8 characters' do
  visit new_user_path

  fill_in 'Email', with: 'xajler@gmail.com'
  fill_in 'Password', with: '123'
  fill_in 'Confirm Password', with: '123'
  fill_in 'Full Name', with: 'Kornelije Sajler'
  click_button 'Sign Up'

  current_path.should == signup_path
  page.should have_content "Password is too short (minimum is 8 characters)"
end
```

The test applies the Passwords that are of 3 characters long and we expect to get right message
that is telling the right expectations of Password in this case minimum of 8 characters.

This was easy, all tests should pass still:

    Finished in 2.07 seconds
    17 examples, 0 failures

> Note:
>
> If you using _Guard_ as I do, for running test, you may need to enter `r` command to
> _Guard_ console to reload all, only if your tests are passing and they should not be or vice versa!

And with this test we have test for all User Create problems, now we'll add test for User Update.

## Valid Update Test

Essentially, the valid update only lets to change User `password` and `full_name`. Create
new `context` called "_PUT users/:id_" and new `it` block "_valid user update_":

```ruby
context 'PUT users/:id' do
  it 'valid user update' do
    user = create :user
    visit edit_user_path user

    find_field('Email').value.should == 'xajler@gmail.com'
    find_field('Full Name').value.should == 'Kornelije Sajler'

    fill_in 'Email', with: 'xajler@gmail.com'
    fill_in 'Password', with: 'aoeuidht'
    fill_in 'Confirm Password', with: 'aoeuidht'
    fill_in 'Full Name', with: 'Kornelije Sajler - xajler'
    click_button 'Update User'

    current_path.should == edit_user_path(user)
    page.should have_content 'The User is successfully updated!'
  end
end
```

First we create a User from factory and save it as variable, because we are simulating edit,
so we need the User id when we want to edit him. This is why we pass the `user` variable to the
`edit_user_path`.

Before we fill in the User Update Form we shall test it if the Form fields are actually binded with
data from User that will be updated.

OK, same thing as for new/create it will fail, first, because there is no method `edit` in
`UsersController`.

```
1) Users POST /users PUT users/:id valid user update
   Failure/Error: visit edit_user_path #user
   ActionController::RoutingError:
     No route matches {:action=>"edit", :controller=>"users"}
```

To pass this error lets create the `edit` method:

```ruby
def edit
  @user = User.find params[:id]
end
```

So we need the User to get him from the given _params id_ so that we can have it in
instance variable `@user` and share it to the View and then fill the form with existing
User data.

The test should still fail, because there is no edit View Template:

```
1) Users POST /users PUT users/:id valid user update
   Failure/Error: visit edit_user_path user
   ActionView::MissingTemplate:
     Missing template users/edit, application/edit with {:locale=>[:en], :formats=>[:html], :handlers=>[:erb, :builder, :coffee, :haml]}. Searched in:
       * "/Users/xajler/src/rb/just-todo-it/app/views"
```

Long time ago when we are creating View Template for Sign Up we've created `new.html.haml`
and also separated form to partial `_form.html.haml` so it can be shared with Edit User View.

Lets create `edit.html.haml` View page that calls the Form partial and passing to it the
`@user` so that the Form will display User data:

```haml
%h1 Update User

= render partial: 'form', locals: { user: @user }
```

The test will fail because there is no button "Update User", and this is because we hard-coded
the "Sign Up" text to submit element in the form partial `_form.html.haml`. We will fix it now,
not very pretty but it will work:

Change `submit` in `_form.html.haml` from:

```haml
= f.submit 'Sign Up'
```
To:

```haml
= f.submit @user.id ? 'Update User' : 'Sign Up'
```

The test should still fail because we are after all trying to update, and there in no
`update` method it `UsersController`:

```
1) Users POST /users PUT users/:id valid user update
   Failure/Error: click_button 'Update User'
   AbstractController::ActionNotFound:
     The action 'update' could not be found for UsersController
```

So, we shall impelment the `update` method for the `UsersController`:

```ruby
def update
  @user = User.find params[:id]

  if @user.update_attributes params[:user]
    flash[:notice] = 'The User is successfully updated!'
    redirect_to edit_user_path
  end
end
```

First we getting User from the database from given User Id, and then call `update_atrributes`
from the data send via Form as `params[:user]`. Set the flash notice that User is updated
and redirect to Update User page where we can see that User is actually updated.

And at last the test is passing:

    Finished in 2.3 seconds
    18 examples, 0 failures

The only problem here is that to Update User the `password`s needed to be entered every
time. There are better solutions, like if they are blank, to ignore them on update, instead
currently it will be validated. And blank password is not allowed!

## Password Mismatch

We will not try all the tests that we tried for User Sign Up, but only one. If you remember,
this was the only test that was needed a code, others were just a proof of existing code,
that is more about User Model than actual User View, but we've needed to run them all to
ensure that flash messages are proper and expected.

```ruby
it 'invalid when passwords mismatch' do
  user = create :user
  visit edit_user_path user

  fill_in 'Email', with: 'xajler@gmail.com'
  fill_in 'Password', with: 'aoeuidht'
  fill_in 'Confirm Password', with: 'aoeu'
  fill_in 'Full Name', with: 'Kornelije Sajler'
  click_button 'Update User'

  current_path.should == edit_user_path(user)
  page.should have_content "Password doesn't match confirmation"
end
```

And the test fails, complaining about View, but actually we need the code in Controller,
to send message of what went wrong to User Update Form:

```ruby
def update
  @user = User.find params[:id]

  if @user.update_attributes params[:user]
    flash[:notice] = 'The User is successfully updated!'
    redirect_to edit_user_path
  else
      flash[:error] = @user.errors.full_messages[0]
      redirect_to edit_user_path
  end
end
```

After adding the else clause to `update` method that handles all errors, the test
should pass:

    Finished in 2.6 seconds
    19 examples, 0 failures

## Ensure Email is not changed after User creation

The only test that is left is to ensure that `email` given on create should never ever
be possible to change!

In this test we shall try to change the email of existing User and it needs to be the same
after update, even though the `email` sent to update is quite different from the one when
User is created:

```ruby
it 'keeps the User Email intact while other fields do change' do
  user = create :user
  visit edit_user_path user

  find_field('Email').value.should == 'xajler@gmail.com'
  find_field('Full Name').value.should == 'Kornelije Sajler'

  fill_in 'Email', with: 'xxx@example.com'
  fill_in 'Password', with: 'aoeuidht'
  fill_in 'Confirm Password', with: 'aoeuidht'
  fill_in 'Full Name', with: 'Kornelije Sajler - xajler'
  click_button 'Update User'

  current_path.should == edit_user_path(user)
  find_field('Email').value.should == 'xajler@gmail.com'
  find_field('Full Name').value.should == 'Kornelije Sajler - xajler'
end
```

The above test ensures that the `email` after creation will stay intact, while other fields
will change, there will be no errors having different `email`. The `email` for update will
be silently ignored.

The implementation code of this validation is actually in User Model,
making the `email` as `attr_readonly` that we did in previous post, this test only ensures
that validation is implemented from View perspective.

And now all 20 test should pass, and we use RSpec format to show them all:

```text
User
  #email must not ever change after it is created
  is valid
  is invalid
    when required #email is not given
    when #email format is not valid
    when #email is not unique
    when required #full_name is not given
    when required #password is not given
    when #password is not at least 8 characters

Users
  GET /signup
    displays the sign up page
  PUT users/:id
    valid user update
    invalid when passwords mismatch
    keeps the User Email intact while other fields do change
  POST /users
    creates and saves the valid user
    not saving invalid user
      when passwords mismatch
      when email is not unique
      when full name is blank
      when password is blank
      when email is blank
      when password is less than 8 characters
  GET /users/new
    displays the create new user page

Finished in 2.86 seconds
20 examples, 0 failures
```

## Users Controller

The whole Users Controller `app/controllers/users_controllers.rb` after this post should look like this:

```ruby app/controllers/users_controllers.rb
class UsersController < ApplicationController
  def new
    @user = User.new
  end

  def create
    @user = User.new params[:user]

    if @user.save
      flash[:notice] = 'The User is successfully saved!'
      redirect_to signup_path
    else
      flash[:error] = @user.errors.full_messages[0]
      redirect_to signup_path
    end
  end

  def edit
    @user = User.find params[:id]
  end

  def update
    @user = User.find params[:id]

    if @user.update_attributes params[:user]
      flash[:notice] = 'The User is successfully updated!'
      redirect_to edit_user_path
    else
        flash[:error] = @user.errors.full_messages[0]
        redirect_to edit_user_path
    end
  end
end
```

## Users Request Spec (Tests)

The whole Users Request Spec `spec/requests/users_spec.rb` after this post should look like this:

```ruby spec/requests/users_spec.rb
require 'spec_helper'

describe 'Users' do
  context 'GET /users/new' do
    it 'displays the create new user page' do
      visit new_user_path

      page.should have_content 'Email'
      page.should have_content 'Full Name'
      page.should have_content 'Password'
      page.should have_content 'Confirm Password'
      page.has_field? 'email'
      page.has_field? 'full_name'
      page.has_field? 'password'
      page.has_field? 'password_confirmation'
      page.has_button? 'Sign Up'
    end
  end

  context 'GET /signup' do
    it 'displays the sign up page' do
      visit signup_path

      page.should have_content 'Email'
      page.should have_content 'Full Name'
      page.should have_content 'Password'
      page.should have_content 'Confirm Password'
      page.has_field? 'email'
      page.has_field? 'full_name'
      page.has_field? 'password'
      page.has_field? 'password_confirmation'
      page.has_button? 'Sign Up'
    end
  end

  context 'POST /users' do
    it 'creates and saves the valid user' do
      visit new_user_path

      fill_in 'Email', with: 'xajler@gmail.com'
      fill_in 'Password', with: 'x1234567'
      fill_in 'Confirm Password', with: 'x1234567'
      fill_in 'Full Name', with: 'Kornelije Sajler'
      click_button 'Sign Up'

      current_path.should == signup_path
      page.should have_content 'The User is successfully saved!'
    end

    context 'not saving invalid user' do
      it 'when passwords mismatch' do
        visit new_user_path

        fill_in 'Email', with: 'xajler@gmail.com'
        fill_in 'Password', with: 'x1234567'
        fill_in 'Confirm Password', with: 'x123'
        fill_in 'Full Name', with: 'Kornelije Sajler'
        click_button 'Sign Up'

        current_path.should == signup_path
        page.should have_content "Password doesn't match confirmation"
      end

      it 'when email is blank' do
        visit new_user_path

        fill_in 'Email', with: ''
        fill_in 'Password', with: 'x1234567'
        fill_in 'Confirm Password', with: 'x1234567'
        fill_in 'Full Name', with: 'Kornelije Sajler'
        click_button 'Sign Up'

        current_path.should == signup_path
        page.should have_content "Email can't be blank"
      end

      it 'when password is blank' do
        visit new_user_path

        fill_in 'Email', with: 'xajler@gmail.com'
        fill_in 'Password', with: ''
        fill_in 'Confirm Password', with: ''
        fill_in 'Full Name', with: 'Kornelije Sajler'
        click_button 'Sign Up'

        current_path.should == signup_path
        page.should have_content "Password digest can't be blank"
      end

      it 'when full name is blank' do
        visit new_user_path

        fill_in 'Email', with: 'xajler@gmail.com'
        fill_in 'Password', with: 'x1234567'
        fill_in 'Confirm Password', with: 'x1234567'
        fill_in 'Full Name', with: ''
        click_button 'Sign Up'

        current_path.should == signup_path
        page.should have_content "Full name can't be blank"
      end

      it 'when email is not unique' do
        create :user
        visit new_user_path

        fill_in 'Email', with: 'xajler@gmail.com'
        fill_in 'Password', with: 'x1234567'
        fill_in 'Confirm Password', with: 'x1234567'
        fill_in 'Full Name', with: 'Kornelije Sajler'
        click_button 'Sign Up'

        current_path.should == signup_path
        page.should have_content 'Email has already been taken'
      end

      it 'when password is less than 8 characters' do
        visit new_user_path

        fill_in 'Email', with: 'xajler@gmail.com'
        fill_in 'Password', with: '123'
        fill_in 'Confirm Password', with: '123'
        fill_in 'Full Name', with: 'Kornelije Sajler'
        click_button 'Sign Up'

        current_path.should == signup_path
        page.should have_content "Password is too short (minimum is 8 characters)"
      end
    end
  end

  context 'PUT users/:id' do
    it 'valid user update' do
      user = create :user
      visit edit_user_path user

      find_field('Email').value.should == 'xajler@gmail.com'
      find_field('Full Name').value.should == 'Kornelije Sajler'

      fill_in 'Email', with: 'xajler@gmail.com'
      fill_in 'Password', with: 'aoeuidht'
      fill_in 'Confirm Password', with: 'aoeuidht'
      fill_in 'Full Name', with: 'Kornelije Sajler - xajler'
      click_button 'Update User'

      current_path.should == edit_user_path(user)
      page.should have_content 'The User is successfully updated!'
    end

    it 'invalid when passwords mismatch' do
      user = create :user
      visit edit_user_path user

      fill_in 'Email', with: 'xajler@gmail.com'
      fill_in 'Password', with: 'aoeuidht'
      fill_in 'Confirm Password', with: 'aoeu'
      fill_in 'Full Name', with: 'Kornelije Sajler'
      click_button 'Update User'

      current_path.should == edit_user_path(user)
      page.should have_content "Password doesn't match confirmation"
    end

    it 'keeps the User Email intact while other fields do change' do
      user = create :user
      visit edit_user_path user

      find_field('Email').value.should == 'xajler@gmail.com'
      find_field('Full Name').value.should == 'Kornelije Sajler'

      fill_in 'Email', with: 'xxx@example.com'
      fill_in 'Password', with: 'aoeuidht'
      fill_in 'Confirm Password', with: 'aoeuidht'
      fill_in 'Full Name', with: 'Kornelije Sajler - xajler'
      click_button 'Update User'

      current_path.should == edit_user_path(user)
      find_field('Email').value.should == 'xajler@gmail.com'
      find_field('Full Name').value.should == 'Kornelije Sajler - xajler'
    end
  end
end
```

## Conclusion

In this post we made sure that User Model Validation that is tested through actual User
Sign Up and Update Views. And all aspects of Validation are tested in integration Broweser
Tests simulated with _Capybara_.

In next post we shall finally tackle the Login page and implementing Authentication for
Application.


## Code

The code is hosted on GitHub and can be cloned from the [xajler/just-todo-it](https://github.com/xajler/just-todo-it).

> Github xajler/just-todo-it commits for this post:
>
> [The User Create and Update Integration Tests, Users Controller and new Edit View Template.](https://github.com/xajler/just-todo-it/commit/ebbd2b0e48b3b6d94571f8d593d8d194bd6aec46)
>
> [A PUT tests are now top context, not child of POST context tests.](https://github.com/xajler/just-todo-it/commit/4da7186bd475e6465f35ee404b2c50b4bbfdf996)
