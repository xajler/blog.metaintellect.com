---
layout: post
title: "Xcode 4.5 Workspace, Empty Project, CocoaPods and Kiwi BDD Test Framework Setup"
date: 2012-11-27 21:57
comments: true
categories: [xcode, cocoapods, kiwi, BDD, testing]
keywords: Xcode, Xcode 4.5, CocoaPods, Kiwi, BDD, Testing
description:
---

This _Xcode_ series of post will include:

 * Create _Xcode Workspace_.
 * Add Empty _Project_ to _Workspace_.
 * Add new _Target_ as the `Universal` App to _Project_.
 * An overview on Xcode Testing.
 * Install the _CocoaPods_.
 * Create a `Podfile` and add _Kiwi_ pod into it.
 * Create new _Target_ for Kiwi Testing to _Project_.
 * Add Test _Target_ Build settings.
 * Create First _Kiwi_ BDD Test.
 * Explore _Kiwi_ Testing Framework.



## Testing in Xcode

It is really good to see in _Xcode_ default template for _Project_ or better to say _Target_
that there is a option to include Tests in to App. Even though I'm not a _iOS_ Developer, from
what I saw (mostly on _GitHub_) there is really poor or not existing usage of Tests for _iOS_
projects.

The default Testing Framework used in _Xcode_ is _OCUnit_ or _SenTestingKit_ which is not
really readable, and using it with some Mocking Frameworke makes it even uglier. And also
it is tipical Assert Testing Framework.

Me liking the _Ruby_ and all Testing goodness that it brings, much more like the _Spec_ Testing
Frameworks that are hiding behind the word of _BDD_. Even for the _TDD_ Unit Tests, I still like to use
_BDD Specs_ even though they should be for different things. But it simplicity and mostly
readability wins, at least in my eyes, against boring Assertive Tests.

There are few _BDD_ Test Frameworks around `Objective-C`, but I choose the [Kiwi](https://github.com/allending/Kiwi).
And beleive it or not there is a book about _Kiwi_ and much more it is a great book!!!
The book is [Test Driving iOS Development with Kiwi](http://editorscut.com/Books/001kiwi/001kiwi-details.html) by
[Daniel H. Steinberg](http://dimsumthinking.com/) and it can be
[purchased through iTunes](https://itunes.apple.com/us/book/test-driving-ios-development/id502345143?mt=11&ign-mpt=uo%3D4).

The [Test Driving iOS Development with Kiwi](http://editorscut.com/Books/001kiwi/001kiwi-details.html) is not
only good book to learn about _Kiwi_ but also great resource for _BDD_ and also Testing in general.
If you're frustrated by default _OCUnit_ Testing, buy a [Test Driving iOS Development with Kiwi](http://editorscut.com/Books/001kiwi/001kiwi-details.html) and you will not going to regret it!
