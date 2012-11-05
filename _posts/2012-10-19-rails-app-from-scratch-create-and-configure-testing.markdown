---
layout: post
title: "Rails App from Scratch: Create and Configure for Testing"
date: 2012-10-19 06:28
comments: true
categories: [rails, ruby, TDD, rspec, guard, sprok, pry, git, github]
keywords: rails, ruby, TDD, rspec, guard, sprok, pry, git, github
description: The Rails 3 App from Scratch, creating and configuring TDD testing, including - RSpec, guard, sprok. Setting up Pry, and git as source control.
---


This blog post will be one many showing how to create _Todo_ the _Rails 3_ application.
The first post will be creating the Rails application and setting up the testing environment.

 The app will be called _Just ToDo it_, just as famous
 <a href="https://en.wikipedia.org/wiki/Just_Do_It_(Nike)" target="_blank">Nike slogan</a>. And also Gods of DNS where
 good to me so the _Domain_ `justtodoit.com` is free so I bought it and the final stage of this
 Rails posts will be deployment to _VPS_ and pointing to the domain [JustToDoIt][1]
 (Currently displays my domain [metaintellect][2]).

## Create application

The command to create new Rails application and omitting default testing framework _Unit::Test_
with switch `-T` or longer version is `--skip-test-unit`

```bash
$ rails new JustToDoIt -T
create
create  README.rdoc
create  Rakefile
create  config.ru
create  .gitignore
create  Gemfile
create  app
...
```

I had capitalized `JustToDoIt` before, because the name is used as Ruby class and Pascal case is convention
for Ruby classes.

Then rename folder to `just-todo-it` to be more in *nix folder naming convention:

```bash
$ mv JustToDoIt just-todo-it
```

The fun starts when the directory is changed to Rails app directory:

```bash
$ cd just-todo-it
```

## Text editor

I'll use _vim_ as a default text editor, for _TextMate_ use `mate` and for the _Sublime Text 2_ use
`subl` terminal commands for editing files instead of `vim`.

## Gemfile

First open the `Gemfile`, we need to add some _gems_ that will be used in the app and also
for testing:

```bash
$ vim Gemfile
```
Edit it to include this gems:

```ruby Gemfile
source 'https://rubygems.org'

gem 'rails'
gem 'bcrypt-ruby'
gem 'unicorn'
gem 'haml'
gem 'thin'
gem 'pg'

group :test, :development do
  gem 'sqlite3'
  gem 'rspec-rails'
  gem 'pry'
  gem 'factory_girl_rails'
  gem 'database_cleaner'
  gem 'awesome_print'
  gem 'capybara'
  gem 'rb-fsevent', :require => false if RUBY_PLATFORM =~ /darwin/i
  gem 'guard-rspec'
  gem 'spork'
  gem 'guard-spork'
end

group :assets do
  gem 'sass-rails',   '~> 3.2.3'
  gem 'coffee-rails', '~> 3.2.1'
  gem 'uglifier', '>= 1.0.3'
end

gem 'jquery-rails'
```

After changing the `Gemfile` run _bundler_ to update and download entered _gems_:

```bash
$ bundle install
```

### Main and Production Ruby Gems

* [rails][3] - The latest one `3.2.8` for this time of writing.
* [bcrypt-ruby][4] - Needed for password hashing.
* [unicorn][5] - For production, it will run as [Rack][6] HTTP Server.
* [haml][7] - My favorite View rendering engine.
* [thin][8] - _Thin_ as local server instead of default _Webrick_.
* [pg][9] - My default database for production usage.

### Test and Development Ruby Gems

* [sqlite3][10] - The database used for development and testing environments.
* [rspec-rails][11] - RSpec as default testing framework.
* [pry][12] - Using as default _Interactive Ruby_ console instead of `irb`. Needs some configuration to be hooked as `rails console`.
* [factory_girl_rails][13] - The testing factory framework, used instead of the default _Fixtures_.
* [database_cleaner][14] - Used to speed-up tests, in my case to encapsulate the tests into db transaction.
* [awesome_print][15] - Used by _Pry_ to pretty prints Ruby objects in full color exposing their internal structure with proper indentation.
* [capybara][16] - for simulating the web interaction in the tests.
* [guard-rspec][17] - To refresh and run the tests upon saving via [rb-fsevent][18].
* [spork][19] - The server to speed up tests, how?, see provided link.
* [guard-spork][20] - Refreshes the spork server on changes, so that we don't need to.


## Testing configuration

### RSpec

_RSpec_ will be used as the test framework for the `Just ToDo it` app.

Run generator to install _RSpec_ to _Rails_:

```bash
$ rails g rspec:install
create  .rspec
create  spec
create  spec/spec_helper.rb
```
### Guard

```bash
$ bundle exec guard init
rspec guard added to Guardfile, feel free to edit it
spork guard added to Guardfile, feel free to edit it
```

Configure `Guardfile` set the _Spork_ on top and _RSpec_ in bottom:

```ruby Guardfile
guard 'spork', :cucumber_env => { 'RAILS_ENV' => 'test' }, :rspec_env => { 'RAILS_ENV' => 'test' } do
  watch('config/application.rb')
  watch('config/environment.rb')
  watch(%r{^config/environments/.+\.rb$})
  watch(%r{^config/initializers/.+\.rb$})
  watch('spec/spec_helper.rb')
  watch(%r{^spec/support/.+\.rb$})
end

guard 'rspec', cli: "--drb" do
  ...
end
```

### Spork

Bootstrap the _Spork_:

```bash
$ spork --bootstrap
Using RSpec
Bootstrapping /Users/xajler/src/rb/just-todo-it/spec/spec_helper.rb.
Done. Edit /Users/xajler/src/rb/just-todo-it/spec/spec_helper.rb now with your favorite text editor and follow the instructions.
```
### RSpec Helper

Edit RSpec helper:

```bash
$ vim spec/spec_helper.rb
```
And edit to include this content:

```ruby spec_helper.rb
require 'rubygems'
require 'spork'
require 'database_cleaner'

Spork.prefork do
  ENV["RAILS_ENV"] ||= 'test'
  require File.expand_path("../../config/environment", __FILE__)
  require 'rspec/rails'
  require 'capybara/rspec'

  Dir[Rails.root.join("spec/support/**/*.rb")].each {|f| require f}

  DatabaseCleaner.strategy = :truncation

  RSpec.configure do |config|
    config.mock_with :rspec
    config.include FactoryGirl::Syntax::Methods
    config.use_transactional_fixtures = true
    config.infer_base_class_for_anonymous_controllers = false
    config.order = "random"
   end
end

Spork.each_run do
  FactoryGirl.reload
  DatabaseCleaner.clean
end
```

It uses _Spork_ server and the aim is to have most things in `prefork` block where is
stuff run on load of _Spork_.

In `each_run` block we want put only necessary things, because it runs each time,
we are now having only reloading of _Factory Girl_ factories, but maybe we will add something
from `prefork` if we would have some troubles with testing data.

_DatabaseCleaner_ is used to start, on before and clean it, on after running.
The strategy used for _DatabaseCleaner_ is transaction, meaning to rollback
changes after the transaction queries are finished.

### Run Guard

The testing environment is now configured, the _Guard_ can be run:

```bash
$ guard
uard could not detect any of the supported notification libraries.
Guard is now watching at '/Users/xajler/src/rb/just-todo-it'
Starting Spork for RSpec
Using RSpec
Preloading Rails environment
Loading Spork.prefork block...
Spork is ready and listening on 8989!
Spork server for RSpec successfully started
Guard::RSpec is running
Running all specs
Running tests with args ["--drb", "-f", "progress", "-r", "/Users/xajler/.rbenv/versions/1.9.3-p194/lib/ruby/gems/1.9.1/gems/guard-rspec-2.1.0/lib/guard/rspec/formatter.rb", "-f", "Guard::RSpec::Formatter", "--out", "/dev/null", "--failure-exit-code", "2", "spec"]...
No examples found.


Finished in 0.11748 seconds
0 examples, 0 failures

Randomized with seed 7715

Done.
```

or use `bundle exec guard` to remove displayed warning.

To exit or stop the `guard` command use `Ctrl+C`.

## Pry as Rails Console

And for the end we will set _Pry_ as our default _Interactive Ruby_ console.

Open the `development.rb`:

```bash
$ vim config/enironments/development.rb
```
At the end of source file add code:

```ruby
silence_warnings do
  require 'pry'
  IRB = Pry
end
```

Try it out with `.pwd` and close the _Pry_ with `exit` command:

```bash
$ rails c
Loading development environment (Rails 3.2.8)
1.9.3 (main):0 > .pwd
/Users/xajler/src/rb/just-todo-it
1.9.3 (main):0 > exit
```

## Commit Source

First remove the `README.rdoc` file and create markdown `README.md`:

```bash
$ rm README.rdoc
$ vim README.md
```

Add simple description:

```text README.md
The simple ToDo Rails App!
```

### Initialize

Initialize the _git_ repository:

```bash
$ git init
Initialized empty Git repository in /Users/xajler/src/rb/just-todo-it/.git/
```

### Status

See the status:

```bash
$ git status
# On branch master
#
# Initial commit
#
# Untracked files:
# .gitignore
# .rspec
# Gemfile
# Gemfile.lock
# Guardfile
# README.md
# Rakefile
# app/
# config.ru
# config/
# db/
# doc/
# lib/
# log/
# public/
# script/
# spec/
# vendor/
nothing added to commit but untracked files present
```
### Add

Then add all files:

```bash
$ git add .
# On branch master
#
# Initial commit
#
# Changes to be committed:
# new file:   .gitignore
# new file:   .rspec
# new file:   Gemfile
...
```

### Commit

Commit the files to local repository:

```
$ git commit -m 'Initial Commit. Created initial Rails app, added all needed Gems, testing configured'
[master (root-commit) e4517a6] Initial Commit. Created initial Rails app, added all needed Gems, testing configured
 38 files changed, 1086 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 .rspec
 create mode 100644 Gemfile
```


### Set Github Remote

The app will be on _Github_. So after the new repository is created on _Github_, we can add
remote to the local repository:

```bash
$ git remote add origin git@github.com:xajler/just-todo-it.git
```

### Push to the Github

After we add remote, it is now safe to push changes to _Github_ remote repository:

```
$ git push -u origin master
Counting objects: 63, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (47/47), done.
Writing objects: 100% (63/63), 23.32 KiB, done.
Total 63 (delta 2), reused 0 (delta 0)
To git@github.com:xajler/just-todo-it.git
 * [new branch]      master -> master
Branch master set up to track remote branch master from origin.
```

## Conclusion

It this first part we have created a `JustToDoIt` Rails application.

And because we shall use _TDD_ (Test Driven Development) to drive this app, we first
configure the testing environment including:

* RSpec
* Factory Girl
* Database Cleaner
* Guard
* Spork

And for the end we setup the _Pry_ to be a default for _Rails_ console and commit the
source to the _Github_ repository [xajler/just-todo-it][22].

In second post we shall go with the creating the app logic in _TDD_ way!

## Code

The code is hosted on GitHub and can be cloned from the [xajler/just-todo-it][22].

> Github xajler/just-todo-it commit for this post:
>
> [Initial Commit. Created initial Rails app, added all needed Gems, testing configured][21]

[1]: http://justtodoit.com
[2]: http://metaintellect.com
[3]: http://rubyonrails.org/
[4]: https://github.com/codahale/bcrypt-ruby
[5]: https://github.com/defunkt/unicorn
[6]: https://github.com/rack/rack
[7]: http://haml.info/
[8]: http://code.macournoyer.com/thin/
[9]: https://bitbucket.org/ged/ruby-pg
[10]: https://github.com/luislavena/sqlite3-ruby
[11]: https://www.relishapp.com/rspec/rspec-rails/docs
[12]: http://pryrepl.org/
[13]: https://github.com/thoughtbot/factory_girl/blob/master/GETTING_STARTED.md
[14]: https://github.com/bmabey/database_cleaner
[15]: https://github.com/michaeldv/awesome_print
[16]: http://jnicklas.github.com/capybara/
[17]: https://github.com/guard/guard-rspec
[18]: https://github.com/thibaudgg/rb-fsevent
[19]: https://github.com/sporkrb/spork
[20]: https://github.com/guard/guard-spork
[21]: https://github.com/xajler/just-todo-it/commit/e4517a6390f912850c63cc60085cbd57770d22ed
[22]: https://github.com/xajler/just-todo-it
