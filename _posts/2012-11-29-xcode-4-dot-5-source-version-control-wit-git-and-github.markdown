---
layout: post
title: "Xcode 4.5: Source Version Control wit Git and Github"
date: 2012-11-29 04:30
comments: true
categories: [xcode, git, github]
keywords: Xcode, Xcode 4.5, Version Control, Git, Github
description: Managing the git Source Version Control in Xcode 4.5 and setting the remotes to Github.
---

OK, so we created the skeleton of App. While creating the _Xcode Project_ we could add the project
to _Git_ repository, but we didn't, we shall do it manually and finally at the end we shall
push it to the _Github_ hosted repository.


## Git Repository

Firstly we really want to add a proper _Xcode_ and _Objective-C_ `.gitignore` file. Here is the
_Gist_ link to one proper [.gitingnore](https://gist.github.com/3786883). Download and copy
it to the App root, before doing the first commit or use `wget` as I did!

Since we didn't create the _Git_ repository when the _Xcode_ was offering. We shall create
repository, add `.gitignore` and create initial commit:

```
wget https://raw.github.com/gist/3786883/93e8364b202f691abc4cf3e22001a45e799011cc/.gitignore
git init .
git add .
git status
git commit -m 'Initial commit'
```

{% img /images/02-01.png %}

> Note
>
> I would recommend creating repository without _Xcode_ help. Because it will create no `.gitignore`
> and by it will commit a lot of junk to repository.
>
> Folders like `xcuserdata` will change frequently and polluting your
> repository with not needed files and even more in teams, it can overwrite another developer's
> `xcuserdata`  every time with not wanted user data from other developer!

## XCode Organizer

The _Xcode Organizer_ is where the _Repositories_ can be viewed.

> Note
>
> If you had a _Xcode_ open during the creation of repository through Terminal, reopen the
> the _Xcode_ because it will not detect the repository in _Organizer_.

To open _Organizer_ go to `Window > Organizer` or with keyboard shortcut **⇧ ⌘ 2**.
There should be a repository _Casa_:

{% img /images/02-02.png %}

## Add Github Remote

To add the _Github_ remote so that we can push the commits to it, we first need to create
a _Github_ repository, I've already created repository [casa-app-xcode](https://github.com/xajler/casa-app-xcode).

Let's add this remote with _Xcode_:

* Go to _Organizer Repositories_.
* Select _Casa_ repository.
* In _Casa_ repository select _Remotes_.
* Hit the `Add Remote` circled plus button.
* For _Remote Name_ enter: `casa-app-xcode`.
* For _Location_ enter: `https://github.com/xajler/casa-app-xcode`.

Ignore the git username and password if you have _Github SSH_ set up.

{% img /images/02-03.png %}

## Push changes to Github


* Choose `File > Source Control > Push...`.
* Wait availe for first time.

{% img /images/02-04.png %}

The commit is pushed to _Githb_.

{% img /images/02-05.png %}

## Commiting changes with Xcode

From the main app delegate `MIAppDelegate.m` we shall remove the generated comments and
commit it with Xcode:

* Applay changes to the `MIAppDelegate.m` by removing commented code in methods.
* Commit it by going to `File > Source Control > Commit...` or use keyboard **⌥ ⌘ C**.
* Enter the commit message.
* Hit the `Commit 1 File` button.

{% img /images/02-06.png %}

* Push changes to [Github repository](https://github.com/xajler/casa-app-xcode).

## Branching in Xcode

On commit it can be chosen to commit to branch with button `Commit to Branch...`. Where
the branch can be chosen or even newly one created.

Alternatively branches can be created in _Organizer Repositories_ by selecting wanted
repository and selecting _Branches_, and hitting `Add Branch` button.

## Diff in Xcode

Anytime while coding the _Version editor_ can be turned on to see changes with current `HEAD`
but also it is possible to change to past revisions.

_Xcode Version editor_ is really nice and with visuals that helping in better understanding of the code changes.

{% img /images/02-07.png %}

## Conclusion

As I said in last post _Xcode 4_ is great IDE, filled with lots of features. Respect to
have _Git_ as default Version control. The _SVN_ is also supported, but who would want to use
it in 21st Century. But in my opinion the Diff part is the greatest feature of it all!

But with all of this, I'm still not impressed with managing _Git_ with IDE, and I will
still be using my favorite _iTerm2_ to interact with _Git_ and _Github_.

## What's next

So we past the _Xcode_ project creation (from ground up!) and in this post the Version Control
as _Git_ and creating remote to the _Github_.

In next _Xcode_ post we will have an broad overview of _Xcode_ support of Testing. What
Testing Frameworks are available, which is the default _Xcode_ Testing Framework.

Look for the _BDD_ Testing Frameworks and picking one for more in depth examine of it. And also
the available Mocking Frameworks and its usage.
