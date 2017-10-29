---
layout: post
title:  "Youtube, AIR and StageWebView"
date:   2013-01-20 21:35:44 +0200
categories: blog
---

If you want to insert a Youtube video in your Adobe AIR app, you're lucky. The most important thing is to know the player URL:

[http://www.youtube.com/embed/id_video](http://www.youtube.com/embed/id_video)

This URL works in almost all devices and browsers, if your browser don't support flash player an HTML5 player will be used. In order to use the magic URL, we create our own browser with the following code:

{% highlight actionscript %}
var videoView : StageWebView = new StageWebView();
var youtubeId : String = "6yCIDkFI7ew";

videoView.stage = this.stage;

// are you using Starling?
// videoView.stage = Starling.current.nativeStage;
videoView.viewPort = new Rectangle(0,  0,
                                    this.stage.stageWidth,
                                    this.stage.stageHeight);
videoView.loadURL("http://www.youtube.com/embed/"
                                            + youtube_id);
{% endhighlight %}

In mobile devices this address will use the native player with many options like volume, fullscreen and more. Pretty good solution.