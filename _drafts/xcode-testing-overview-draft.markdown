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

## To Buy

[The iOS Apprentice: Learn iPhone and iPad Programming via Tutorials!](http://www.raywenderlich.com/store/ios-apprentice)

## To Watch

[iOS Testing Talks](Running OCUnit & Kiwi Tests on the Command Line)
[Must Read about Xcode 4](http://maniacdev.com/xcode-4-tutorial-and-guide/)

## To Watch

[Tolerant JSON Parsing for iOS](http://www.stewgleadow.com/blog/2012/05/18/tolerant-json-parsing-for-ios/)

## Testing Links

### Running from CommandLine

[Running OCUnit & Kiwi Tests on the Command Line](http://www.stewgleadow.com/blog/2012/02/09/running-ocunit-and-kiwi-tests-on-the-command-line/)
[StackOverflow: Running iOS Test from Command Line](http://stackoverflow.com/questions/5403991/xcode-4-run-tests-from-the-command-line-xcodebuild/10823483#10823483)

[Adding Unit Tests to an existing iOS project with Xcode 4](http://twobitlabs.com/2011/06/adding-ocunit-to-an-existing-ios-project-with-xcode-4/)

##  How to customize UINavigationBar

[How to customize UINavigationBar](http://www.altinkonline.nl/tutorials/xcode/uinavigationbar/customizing-background-tintcolor-and-selected-image/)

## Localization

[More About Localization In iOS Development](http://www.lwxted.com/blog/2012/localization-ios-development/)

[How to force NSLocalizedString to use a specific language](http://stackoverflow.com/questions/1669645/how-to-force-nslocalizedstring-to-use-a-specific-language)
ibtool --generate-strings-file MainStoryboard.strings Main.storyboard
ibtool --strings-file MainStoryboard.strings  --write ../hr.lproj/Main.storyboard Main.storyboard
genstrings -o Base.lproj *.m

### Coverage

[iOS dev: How to get your code coverage right?](http://blog.octo.com/en/ios-development-right-code-coverage/)
[StackOverflow: Code Coverage](http://stackoverflow.com/questions/8459330/code-coverage-on-ios-using-xcode-4-2-on-lion)
[LLVM, Code Coverage and Xcode 4](http://mattrajca.com/blog/2011/08/14/llvm/)
[iOS Code Coverage, Cobertura and Jenkins](http://jonboydell.blogspot.com/2011/11/ios-code-coverage-cobertura-and-jenkins.html)

## Set Storyboard to Empty Application

For an Empty Application project, you have to do two things, after you've added your Storyboard file...

Add a row to your project Info.plist file:

Key: Main storyboard file base name
Value: Storyboard
(or whatever you named your storyboard file)

Delete the contents of application:didFinishLaunchingWithOptions: except return YES;:

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    return YES;
}

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
