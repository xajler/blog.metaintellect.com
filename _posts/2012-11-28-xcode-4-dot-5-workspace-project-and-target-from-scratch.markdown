---
layout: post
title: "Xcode 4.5: Workspace, Project and Target Setup from Scratch"
date: 2012-11-28 10:35
comments: true
comments: true
categories: [xcode]
keywords: Xcode, Xcode 4.5, Xcode Workspace, Xcode Empty Project, Xcode Empty Application
description: Setting up the minimal Xcode 4.5 project from scratch. Including Workspace, Empty Project and Empty Application as Target.
---


In aim to learn more about _Apple_'s Development stack and because I will be creating an
_iPhone_ and will include _iPad_ App as `Universal` App. I'll be going through creation of
the minimal as possible project setup for current stable version of _Xcode 4.5.2_ and _iOS 6_.

## Workspace

So first, what exactly is a _Workspace_, as I remember it was introduced with _Xcode_ 4, and it
is a simply a container for combining multiple projects if they are needed, if not only _Project_
will due. But in our case, we know that we shall have multiple projects, so the _Workspace_ is way to go.

> **Note**
>
> Maybe the desired workflow by _Apple_ was first to create a _Project_ and then if there is
> need for another project to bundle them into _Workspace_. But I wanted show the hierarchical
> structure of _Xcode_ project, so on the top is a _Workspace_ then _Project_.

Create it by:

* First create a folder (with Finder or Terminal), name it whatever the App will be called in my case _Casa_.
* Create _Workspace_ by going to `File > New > Workspace...` or better with the keyboard key **⌃ ⌘ N**.
* Enter desired name, in my case `Casa` and save it to _Casa_ folder already created.

{% img /images/01-01.png %}

* It will create _Workspace_ as folder named `Casa.xcworkspace`.
* Do not close the _Workspace_ or _Xcode_ for now.

> **Geek Tip**
>
> For hard-core coders open _Xcode_ and _Workspace_ from _Terminal_ with `open Casa.xcworkspace`.

## Empty Project

The _Xcode_ project is the thing that is usually what it is created in most examples.
This way _Project_ and the _Target(s)_ will be created together.

But not to get the ahead of myself, we will not use this pattern, since we broke it with
creation of _Workspace_ in first place, we will broke it even more by creating empty project.

Because when you start with the _Empty Project_ you can actually see what _Project_ is, basically
nothing, just another container, that will hold _Targets_.

* Create _Empty Project_ by going to `File > New > Project...` or by keyboard shortcut **⇧ ⌘ N**.
* Something strange will happen, another instance of _Xcode_ will open!!!
* From _iOS_ group choose _Other_ in left pane and in right pane choose _Empty_ as blue _Project_ icon.

{% img /images/01-02.png %}

* Hit the `Next` button, add _Product Name_, in my case, still _Casa_.
* Again hit `Next` button, go to _Casa_ folder, because we want the project to be created there.
* Uncheck the _Create local git repository for this project_, we will set it later.
* And for _Add to_ we want to add it to _Casa Workspace_ so choose _Casa_.

{% img /images/01-03.png %}

* It will close this instance of _Xcode_, but second instance will have a _Workspace_ and _Project_ all together!

{% img /images/01-04.png %}

## Target (Empty Application)

So finally we're getting to the meat of _Xcode_, the _Target_ is what will hold all you code, resources,
settings for build or deployment. If we had not chosen _Empty Project_ in last part,
it would create _Project_ and _Target_ all together, even if we chose the _Empty Application_.

But we are learning all the parts so we shall in this case create a simple as possible _Target_
for _iOS_ - _Empty Application_.

* Make sure the `Casa` _Project_ is selected and at bottom hit a big plus button `Add Target`.
* In _iOS_ group choose _Application_ on left and on the right choose _Empty Application_. Hit `Next`.

{% img /images/01-05.png %}

* For _Product Name_ we will add, you guessed it: _Casa_.
* Add the _Organization Name_ and _Company Identifier_.
* I tend to add _Class Prefix_ (`MI` as _Metaintellect_), because there is no namespacing in `Objective-C`.
* Depending what are you targeting, you'll choose _Devices_, I'm feeling lucky so I'm going with hardest `Universal` (iPhone and iPad support).
* _ARC (Automatic Reference Counting)_ should be left checked!
* _Include Unit Test_ should be unchecked, because we will set it later, from ground up!

{% img /images/01-06.png %}

* Hit `Finish` button and we have a target created.
* Build it by `Project > Build` or with **⌘ B** and we can ship it :)

{% img /images/01-07.png %}

## My Thoughts on Objective-C

So there are many ways to get to _AppStore_ and not using the default _Apple_ language: _Objective-C_.
One of many are _RubyMotion_ (_Ruby_), _MonoTouch_ (_C#_), _PhoneGap_ (_HTML_, _CSS_, _JS_),..., but
_Objective-C_ is still used as preferred language.

There is a lot of criticism on _Objective-C_ language, even though it is a _C_ based language,
like _C++_, _Java_ and _C#_. But those are kind of similar language where transition is very
easy, but transition to _Objective-C_ is awkward, even more than to _JavaScript_. And this
is mostly because _Objective-C_ has a not only strong legacy in _C_ (static and procedural part) but
also in one of the underestimated language _Smalltalk_ (dynamic and object or better message driven part).

And even though the _Objective-C_ looks very awkward, but because it has a legacy from two great
languages that are in my opinion pretty hard and crazy to merge, has my everlasting respect. And I
find my self writing _Objective-C_ better and with more joy than the _Java_ or _C#_.

What I dislike about the _Objective-C_ is that code snippets, examples found on web are mostly so
poorly written, and same for most paid Video Tutorials. So much repetitive code, huge methods,
**copy-paste driven development** everywhere, same as in most _.NET_ (especially VB.NET), _PHP_,
and also in _Rails_ (excluding pure _Ruby_ programmers) examples!

There is quite a lot resources how to create _iOS_ apps but mainly focusing on the features
like Table Views, MapKit, and so on. But it is really hard to find good source on
Design Patterns, refactoring and best coding practices, or how to structure the _iOS_
app into Groups (there are no Folders, files are all in root of _Target_).

But there are quite neat features in _Objective-C_ features like protocols and categories.
I also like the progress or evolution of _Objective-C_ for embracing properties, _ARC_ (Automatic
Reference Counting), Blocks, GCD (Grand Central Dispatch) and new literals for Arrays, Dictionaries,...

And for the end abandoning the old _GCC_ compiler and embracing and involving in creating
_LLVM_ compiler, much faster, and now with really great code inspection built in to _Xcode_...

## My Thoughts on Xcode 4

And there is _Xcode_, the version 3 was too funky too me, especially separate _Interface Builder_
and Apple old-school floating inspectors and controls all around the desktop.

Bet then came the version 4 and even though I really dislike the IDEs, must admit that it is to me
the best IDE. Especially comparing to _Eclipse_ and _Visual Studio_. Responsiveness is great,
integration with _Git_ (even though I still prefer using it by Terminal) and diff viewer
is really what was expected of Apple, also _StoryBoard_ and integrated _Inerface Builder_.
Multi monitor friendly, assistant editor, help integrated, I think you get it, the best IDE I've
worked! Don't get me wrong there is still place for improvements but in my opinion _Apple_ is on the
right track.

## What's Next

In next post we'll examine _Xcode_ SCM (Source Control Management) capabilities using built-in
support for _Git_. Also we'll push this project to _GitHub_, examine _Xcode_ _Git_ Diffing and History.
