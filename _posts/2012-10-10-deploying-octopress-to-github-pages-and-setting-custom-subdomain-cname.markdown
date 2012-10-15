---
layout: post
title: "Deploying Octopress to Github Pages and setting custom subdomain CNAME"
date: 2012-10-10 23:49
comments: true
categories: [octopress, github, github pages]
keywords: octopress, github, github pages
description: Deploying Octopress to Github Pages and setting blog to custom subdomain provided with CNAME to Github Pages.
---

Blogging was really never easy task for me, I started in 2006 with [WordPress](http://wordpress.com) but with short life.
The more about my blogging history can be read in my blog post
[and finally comes time for Blogging!](http://www.learnaholic.me/2009/04/26/hello-world/) on my
old blogging site [Learn-a-holic](http://www.learnaholic.me).

## Blog Engines

So far I was on this blog engines, chronologically:

* [WordPress](http://wordpress.com) (2006 - 2008)
* [Graffiti CMS](https://graffiticms.codeplex.com/) (2008)
* [BlogEngine.NET](http://www.dotnetblogengine.net/) (2009)
* [WordPress](http://wordpress.com) (2009 - 2012)

So far I was always coming back to [WordPress](http://wordpress.com), but I was never actually satisfied in writing
posts in it, everything other than that is plain perfect on [WordPress](http://wordpress.com).
This is why I haven't blogging for nearly two years.

Nowdays I really like [markdown](http://daringfireball.net/projects/markdown/)
or any similar kind of markup language ([textile](http://redcloth.org/textile),
[creole](http://www.wikicreole.org/wiki/Home),...), and
I would prefer writing blog posts with [markdown](http://daringfireball.net/projects/markdown/).
In my case blog posts are more notes than the blog posts,
because they are more technical, and there is lots of code snippets!

## Octopress

[Octopress](http://octopress.org/) is the very easy framework on top of [Jekyll](https://github.com/mojombo/jekyll), and
Jekyll is a blog-aware, static site generator in [Ruby](http://ruby-lang.org).

### Install

The official [Octopress](http://octopress.org/) install is just to clone or fork the
[Octopress repo](https://github.com/imathis/octopress). I've chose to do fork then
clone it from my forked repo.

``` bash
$ git clone git@github.com:xajler/octopress.git
```

> **Notes:**
>
> You'll need to change the clone to your repository, since there is need for Read+Write access!
>
> I'm lying a bit the first attempt was a clone, but afterwards I've chose to fork it, why?,
> you'll need to read about it a little bit later.

## Using Octopress with Rake

The [Octopress](http://octopress.org/) uses Jim Weirich's great make-like build utility
[Rake](https://github.com/jimweirich/rake),
for creating posts/pages, deploying, generating,...

### Rake Install

The [Octopress](http://octopress.org/) needs for the first time to generate the sandbox with theme and placeholder
folder for posts. It can be achieved by running (needs to be in forked/cloned directory):

``` bash
$ rake install
## Copying classic theme into ./source and ./sass
mkdir -p source
cp -r .themes/classic/source/. source
mkdir -p sass
cp -r .themes/classic/sass/. sass
mkdir -p source/_posts
mkdir -p public
```

The command will create `source` and `sass` folders. The `source/_posts` folder is where the markdown
posts will reside.

This is why I forked the [Octopress](http://octopress.org/), so that I can commit the posts as
the markdown files, if I would clone it, I should have the write access to [Octopress](http://octopress.org/),
to commit, this way I have my forked version and still I can always pull changes from
upsteramed original [Octopress](http://octopress.org/), but more of this in separate post.

### Creating first post

Create post with `new_post['post name here']`:

``` bash
$ rake new_post['Deploying Octopress to Github Pages and setting custom subdomain CNAME']
mkdir -p source/_posts
Creating new post: source/_posts/2012-10-10-deploying-octopress-to-github-pages-and-setting-custom-subdomain-cname.markdown
```

By opening it in your favorite editor you'll get:

``` text 2012-10-10-deploying-octopress-to-github-pages-and-setting-custom-subdomain-cname.markdown
---
layout: post
title: "Deploying Octopress to Github Pages and setting custom subdomain CNAME"
date: 2012-10-11 01:01
comments: true
categories:
---
```

Categories can be add in few ways, my way is one-liner:

    categories: [octopress, github, github pages]

### Generating the Site

Add some content to your first post and then run `generate` command that generates whole site:

``` bash
$ rake generate
## Generating Site with Jekyll
directory source/stylesheets/
   create source/stylesheets/screen.css
Configuration from /Users/xajler/src/octopress-orig/_config.yml
Building site: source -> public
Successfully generated site: source -> public
```

### Preview Site

To locally preview generated Site use:

```bash
$ rake preview
Starting to watch source with Jekyll and Compass. Starting Rack on port 4000
[2012-10-11 01:16:08] INFO  WEBrick 1.3.1
[2012-10-11 01:16:08] INFO  ruby 1.9.3 (2012-04-20) [x86_64-darwin12.0.0]
[2012-10-11 01:16:08] INFO  WEBrick::HTTPServer#start: pid=62929 port=4000
Configuration from /Users/xajler/src/octopress-orig/_config.yml

...
```

Then in browser navigate to:

    http://localhost:4000/

The beauty of the `preview` is that _Auto-regenerating_ is enabled by default.
Meaning every save of blog post file will trigger `generate` command making it as live preview.

### Configuring the Octopress

The [Octopress](http://octopress.org/) configuration is in the root `_config.yml` file.

There you can change `url`, `title`, `subtitle`, `permalink`, `twitter account`, `github account`,
and set site comments via [disqus](http://disqus.com/) by providing `disqus_short_name`.

## Deployment

[Octopress](http://octopress.org/) supports [Heroku](http://heroku.com), rsync and
[Github Pages](http://pages.github.com) deployment.
I chose the [Github Pages](http://pages.github.com) because
[Github](http://www.github.com) provide the CNAME changes, so the site can be on your desired domain.

### Github Pages setup

There are two ways of having pages on the [Github](http://www.github.com).

The User/Organization pages `http://username.github.com` and project pages `gh-pages`.

The my way was the User/Organization pages.

#### Setting up the repository on Github

Create new repository on [Github](http://www.github.com) and named it as your User/Organization name plus `.github.com`
in my case was `xajler.github.com`.

#### Configure Octopress for Github Pages

To prepare [Octopress](http://octopress.org/) for deployment to [Github Pages](http://pages.github.com)
run commannd and write `Repository url` (`git@github.com:xajler/xajler.github.com`):

``` bash
$ rake setup_github_pages
Enter the read/write url for your repository
(For example, 'git@github.com:your_username/your_username.github.com)
Repository url: git@github.com:xajler/xajler.github.com
rm -rf _deploy
mkdir _deploy
cd _deploy
Initialized empty Git repository in /Users/xajler/src/octopress-orig/_deploy/.git/
[master (root-commit) ac273d4] Octopress init
 1 file changed, 1 insertion(+)
 create mode 100644 index.html
cd -

---
## Now you can deploy to http://xajler.github.com with `rake deploy` ##
```

This command will change `git remote origin` from [Octopress](http://octopress.org/) forked/cloned url to:

    git@github.com:xajler/xajler.github.com

And also add a `_deploy` folder, actual root of your [Github Pages](http://pages.github.com) repository
(`github.com/xajler/xajler.github.com`).

#### Deployment to Github Pages

The final `rake` command is `deploy`. It will create commit and push to [Github](http://www.github.com)
repo and you'll get notification to your mail.

``` bash
$ rake deploy
## Deploying branch to Github Pages
rm -rf _deploy/index.html

## copying public to _deploy
cp -r public/. _deploy
cd _deploy

## Commiting: Site updated at 2012-10-11 00:14:34 UTC
...
```

After deployment in your mail box will be a notification:

    [xajler.github.com] Page build successful

    Your page has been built. If this is the first time you've pushed, it may take a few minutes to appear, otherwise your changes should appear immediately.

And if you navigate to `http://xajler.github.com` (change xajler to your User/Organization name)
you'll see the post and site online!

## The custom domain via CNAME

The `http://xajler.github.com` is not really the URL that you'll want to have for the blog.
[Github](http://www.github.com) provides the way to customize the domain name.

### Setting custom domain name

To setup the custom domain name that will point to your Github pages, there is need to
create the `CNAME` file in `source` folder. This `CNAME` will be copied to `_deploy` folder
when executing `rake deploy` and will be used by [Github](http://www.github.com) to point to the provided domain.

``` bash
$ vim source/CNAME
```

I've added my subdomain for my domain [metaintellect](http://metaintellect.com).

    blog.metaintellect.com

Deploy with `rake deploy` and it should now be pushed and visible in your
[Github](http://www.github.com) repo (`xajler.github.com`).

## Setting DNS for subdomain

The DNS nameservers for my domain [metaintellect](http://metaintellect.com) are declared on
[dynadot](http://dynadot.com).

Here is my configuration to make the `blog.metaintellect.com` CNAME work with [Github Pages](http://pages.github.com):

{% img https://pbs.twimg.com/media/A44tgDACAAAsDwn.png:large %}

## Conclusion

I hope that I will now blog my notes about so many tehnical IT stuff I do everyday, and
now with markdown and the [Github](http://www.github.com) it is fairly easy and fun!
