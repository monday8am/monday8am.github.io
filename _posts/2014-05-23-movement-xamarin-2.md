---
layout: post
title:  "Movement to Xamarin. Second part"
date:   2014-05-24 21:35:44 +0200
categories: blog
---

The last month I was trying to find enough time to continue [a post I wrote](/blog/2013/08/04/movement-xamarin) almost one year ago about Xamarin and cross platform mobile programming. At this time, I was speaking about my reasons to change from Flash to Xamarin and not to HTML5, that is the natural change for a frontend developer.

From this moment to now Xamarin has evolved in a very fast way. I'll try to summarize the most interesting points related with the the platform development:

### Asynchronous programming support

With the introduction of _async_ and _await_ keywords I have erased the word "callback" from my head. Now, make a request to an API REST and display a progress component while the data is coming is a pleasant task.

### Portable Class Libraries

The concept is very simple: a project that contains code common to all platforms. This code implements only a subset of .Net framework but can be “enhanced” with code on platform projects. Moving as much code as possible to [PCLs](http://blog.stephencleary.com/2012/11/portable-class-library-enlightenment.html) is the best strategy to face a cross-platform project in Xamarin nowadays.

### Mvvmcross Framework

Stuart Lodge has done an amazing job designing, documenting and promoting the Mvvmcross, a _MVVM_ framework. If we have to move the logic to a common place like a PCL, [Mvvmcross](https://github.com/MvvmCross/MvvmCross) is our best friend because not only helps you to create ViewModels, it gives you other patterns implementations like _Service Location_ and _Inversion of Control_, _Messaging_, _Navigation_ or different ways of databinding. And more important, Mvvmcross offers a clear workflow to create an app with modern and well documented patterns, a great starting point for newbies (like me).

### Microsoft partnership

Is not a secret that Microsoft has considered Mono and later Xamarin as part of the family. But the arrival of Nadella as CEO and his focus on mobile and cloud services has opened a new chapter in the relationship between both companies. Xamarin is crucial in Microsoft strategy to use their tools and languages to publish apps on Google and Apple stores. You only have to check the Xamarin popularity in the last BUILD.

Is very hard to speak about the future in a moment **when giants becomes dwarfs** in few time (aka Nokia or Blackberry) but I think Xamarin is setting path to be an stable and time-durable platform.

The key is (of course!) in the language: recently we watched a renascence of Javascripts thanks to Google and its open technologies, maybe we could see the rise of C# helped by Microsoft, .Net Foundation and Unity. In my personal opinion, this is the real goal of a C# lover like Miguel de Icaza because among other things, if C# wins, Xamarin will also win.

#### Related

[Movement to Xamarin](/blog/2013/08/04/movement-xamarin)