---
layout: post
title:  "Fastlane and Gradle for Android development"
date:   2019-01-13 10:43:00 +0100
categories: blog
---

### Automation always saves time

From my experience, in most cases if you create a script to execute (without human intervention) five or six actions, you'll save time. If these actions include passwords, keys, version numbers or they must follow a predefinded order then the time saved is multiplied by four because you're saving the time of fix your mistake too.

This idea, however simple, is easy to forget if the script implementation takes more time than you were planning, or if it is a bit complex. That's why automation should be done in my opinion at any cost, because the cost always will be less than the future time saving.

### Gradle and Fastlane

Automation in Android development includes the one principal and almost obligatory actor Gradle and the luxury guest and co-start fastlane.

Gradle is the do-everything automation tool used by Android studio to compile, test and generate bundles. It supports many languages including Kotlin and it let captures all variables asociated with an Android project, for example, project name, files, location, etc. Android Studio exposes many of his internal Gradle tasks, letting an external actor to combine them from outside.

Fastlane, as Gradle, is an automation tool but more oriented to build and release mobile apps. It includes scripts to interact with both iOS and Android platforms and although the supported language is Ruby (maybe not too familiar to mobile devs) there is a nice team behind that keeps all the lanes (script) up to day and make them easier to use by non-ruby writers.

Considering both tools, an interesting approach for Android is to define small tasks in Gradle files and group them in more general tasks using fastlane. Let's write an example:

{% gist 45a55bd699e3bc710d80deaceaefd4cc %}


# Gradle and Gradle

Why we need to use 2 automation tools and not to use Gradle for everthing? Well, it depend on the context.


### Conclusions



#### Links

- [Reactive Extension portal](http://reactivex.io/)
- [RxSwift](https://github.com/ReactiveX/RxSwift) and [RxKotlin](https://github.com/ReactiveX/RxKotlin)
- [RxBluetoothKit](https://github.com/Polidea/RxBluetoothKit): example of a Rx layer on top of the [CoreBluetooth](https://developer.apple.com/documentation/corebluetooth) API