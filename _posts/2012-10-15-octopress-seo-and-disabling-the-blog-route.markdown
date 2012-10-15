---
layout: post
title: "Octopress SEO and disabling the blog route"
date: 2012-10-15 08:36
comments: true
categories: [octopress, seo]
keywords: octopress, seo
description: Better Octopress SEO with per page keywords and on home page. Redirect to single domain. Removing Octopress default /blog route.
---


The SEO on Octopress by default is moderate, but it can be better, too bad that the `rake new_post`
doesn't generate _Meta description_ and the _keywords_. The other problem is
redirect to single page which is broken since there is _learnaholic.me_ and _www.learnaholic.me_
which can affect site ranking.

The third problem is that Octopress by default has a `<domain>/blog` route path which is awkward
and unnecessary, domain is sufficient enough without `/blog`.

The most SEO fixes are from the [SEO for Octopress,Heroku](http://www.yatishmehta.in/seo-for-octopress)
post by Yatish Mehta.

## Keywords and Description for every page

The provided _keywords_ and _description_ should be a goal for each page, the problem is
that `rake new_post` doesn't generate _keywords_ and _description_, so it should be added
manually.

I've added the _keywords_ and _description_ to all my posts created in this few days
and the post Octopress metadata looks like this:

```
---
layout: post
title: "Make Powershell, SSH Github and git suck less on Windows"
date: 2012-10-12 03:41
comments: true
categories: [powershell, git, github]
keywords: powershell, git, github, windows, ssh key, posh-git, msysgit
description: Installing the msysgit, configuring git, creating SSH keys for Github, customize the Powershell, installing posh-git. Windows suck less after.
---
```

## Home page Keywords and Description

The Octopress by default shows latest post as home page, I choose not to go this way, my
default home page is archive list. So there is no post from where it should include the
_keywords_ and _description_.

### Setting Keywords and Description for Home page in _config.yml

Open the `_config.yml` and add the _kewords_ and _description_ keys:

```
description: Kornelije Sajler (xajler) Learn-a-holic Geek Notes. Human compiled Brainwork by Kornelije Sajler (xajler).
keywords: Kornelije Sajler, xajler, metaintellect, learnaholic, learn-a-holic, coding, programming, Ruby, Ruby On Rails, RSpec TDD, cucumber, jasmine, bacbone.js, postgresql, mongodb
```

### Change head.html template to be aware of Home page SEO

In `.themes/classic/source/_includes/head.html` after meta tag for `author` replace the
current _description_/_keywords_ code with this one:

{% gist 2460469 %}

## Single domain rewrite

To make sure that there is no differnce between _learnaholic.me_ and _www.learnaholic.me_
request, we shall include rewriting with _gem rack-rewrite_.

To Octopress _Gemfile_ add:

```
gem 'rack-rewrite'
```

To top of `config.ru` (before `SinatraStaticServer`) add:

```ruby
ENV['RACK_ENV'] ||= 'development'

ENV['SITE_URL'] ||= 'www.learnaholic.me'

use Rack::Rewrite do
  r301 %r{.*}, "http://#{ENV['SITE_URL']}$&", :if => Proc.new { |rack_env|
      ENV['RACK_ENV'] == 'production' && rack_env['SERVER_NAME'] != ENV['SITE_URL']
    }
  r301 %r{^(.+)/$}, '$1'
end
```

Don't forget to add a require on the top of the `config.ru`:

```ruby
require 'rack-rewrite'
```
Complete `config.ru` should look like this:

```ruby
require 'bundler/setup'
require 'sinatra/base'
require 'rack-rewrite'

# The project root directory
$root = ::File.dirname(__FILE__)

ENV['RACK_ENV'] ||= 'development'

ENV['SITE_URL'] ||= 'www.learnaholic.me'

use Rack::Rewrite do
  r301 %r{.*}, "http://#{ENV['SITE_URL']}$&", :if => Proc.new { |rack_env|
      ENV['RACK_ENV'] == 'production' && rack_env['SERVER_NAME'] != ENV['SITE_URL']
    }
  r301 %r{^(.+)/$}, '$1'
end

class SinatraStaticServer < Sinatra::Base

  get(/.+/) do
    send_sinatra_file(request.path) {404}
  end

  not_found do
    send_sinatra_file('404.html') {"Sorry, I cannot find #{request.path}"}
  end

  def send_sinatra_file(path, &missing_file_block)
    file_path = File.join(File.dirname(__FILE__), 'public',  path)
    file_path = File.join(file_path, 'index.html') unless file_path =~ /\.[a-z]+$/i
    File.exist?(file_path) ? send_file(file_path) : missing_file_block.call
  end

end

run SinatraStaticServer
```

## Set the route without unnecessary /blog route path

The Octopres by defult has a weird _&lt;domain&gt;/blog/2012/10..._ route path, the `blog`
part of URL is totally unnecessary, so I removed it all together.

In `_config.yml` change:

```
permalink: /blog/:year/:month/:day/:title/
category_dir: blog/categories
```

to:

```
permalink: /:year/:month/:day/:title/
category_dir: categories
```

## Conclusion

Improved SEO of the Octopress site:

* Including the _keywords_ and _description_ to each post.
* Home page is now with _keywords_ and _description_, generic for the whole site.
* Redirect to single domain. The _www.learnholic.me_ request will be redirected to _learnaholic.me_.

The cleaner site path by removing the unnecessary `/blog` from the URL route.
