---
layout: post
title:  "Movement to Xamarin and mobile development"
date:   2013-08-04 21:35:44 +0200
categories: blog
---

The reasons that carried up the _Flash platform_ to the decadency are well known: the first strike was from Steve Jobs, then Adobe contributed changing the Flash roadmap many times introducing the mistrust inside the community and finally HTML5 (and the compatibility across modern browsers) did the rest.

Many developers had to reorient their professional careers to new paths. A big group started to use Javascript and CSS in the same way they had used Flash before, doing websites. Another group remained using _AS3_ but based on _Stage3D_, [Starling](https://gamua.com/starling/) and [FeathersUI](https://feathersui.com/), focused on gaming and mobile apps. I did a big jump to use **Xamarin** and **C#** language to develop mobile apps.

### Why Xamarin

* _Concept_: Xamarin has a different approach to multi-platform paradigm: using a common language C# and a good software design you can share files across platforms, around 60-70%. This files could include app logic, data access layers or even business objects.

* _Native UI_: For interface implementation, Xamarin allows you access to native (iOS or Android) APIs, objects and methods to show lists, buttons or navigation bars. Even though you have to develop the interface code every time you add a platform, the native components will give you a big advantage of time and performance.

* _Language_: I don’t like Javascript, I prefer strong-typed languages. Xamarin uses C# 5.0 and includes .Net Framework 4.5 inherited from Mono. Using a popular language like C# gives you access to a huge amount of code samples, libraries and frameworks.

* _Platform code reutilization_: Xamarin lets you "bind" libraries that are written in native code (Java or Objective C).

* _Develpment tools_: The **Xamarin Studio** is based on Monodevelop and offers fast code-completion, interface editor, SVN or Git, facilities to publish on TestFlight and more. It isn't completely mature but has a good future. Besides you have the option to use Visual Studio doing some tricks.

* Xamarin was created by an start-up (named Xamarin too) leadered by Miguel de Icaza, creator and promoter of linux shell Gnome and Mono .Net environment. Personally I think Miguel has a smart vision of future and he is a fantastic developer.

### Drawbacks

Not everything is pink, **its cost is high enough** to think twice before you buy it, specially if you compare it with open solutions more suitables for simple apps. Anyway I think the indie option is really affordable. Besides the cost, Xamarin philosophy includes the use of native APIs, and therefore **you must learn how to develop in iOS or Android** exactly as native developers do, and of course, read and understand Objective C and Java.

### Additional good points

1. Xamarin Studio and C# is a combination used by many Unity3D developers to create their games.

1. [TestCloud](https://www.xamarin.com/test-cloud) is an innovative user interface testing platform offered by Xamarin

1. Xamarin has a [component store](https://components.xamarin.com/)

1. You can reach mobile Windows Phone, iOS, Android and Mac.

1. [Playscript](https://www.zynga.com/blogs/engineering/playscript-compiler-and-runtime-cross-platform-world-2), created by Zinga, is a new language that mixes AS3 and C# and is easy to achieve by Flash developers. The Playscript compiler is integrated into Mono .Net environment and is accessible using Xamarin Studio or Monodevelop. With Playscript you can reuse your old AS3 projects and target mobile using Xamarin platform.