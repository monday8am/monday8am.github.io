---
layout: post
title:  "Fidesmo Android app"
date:   2018-11-10 10:47:00 +0100
categories: project
---

[Fidesmo](https://www.fidesmo.com/) is a company that created a [whole platform](https://developer.fidesmo.com/) to digitize and connect payment cards and other contactless services to smart wearables, and to fill these wearables with digital services. For example: to make it possible for _Mastercard_ to remotely and securely put a credit card information onto a [Fidesmo enabled device](https://www.fidesmo.com/fidesmo/devices/) like a watch, letting customers to use it for contactless payment without buy an expensive _Apple watch_.

### Architecture

Inside the platform, the Android app acts as a secure bridge between the server side logic related with the service provided and the secure element that will hold this service (_this image is property of Fidesmo_).

![Screenshot]({{ "/assets/projects/fidesmo-architecture.png" | absolute_url }})

The app opens a _https_ channel to the server together with a _NFC_ or _BLE_ channel to the device, letting the data and commands flows through the it. The  service delivery involves many synchronous and asynchronous operations, so, in order to tame the complexity a [_Redux_ like framework](https://github.com/ReKotlin/ReKotlin) is used inside.

### UX/UI

One of the main challenges is that the app is used by engineers testing experimental services, customers using production services and by random users that buy and test Fidesmo enabled devices. The UI/UX design should be enough flexible to hold a broad spectrum of user-cases.

![Screenshot]({{ "/assets/projects/fidesmo-ui.png" | absolute_url }})

### Ecosystem

The app is integrated into a continuos integration pipeline built over _Travis_, _Fabric_ and _fastlane_ that provides a fast release pace of new features and bugs fixed.

### Services

- Frontend development and maintenance
- Content creation
- UX/UI design collaboration

### Related links

- [Fidesmo app at Google Play](https://play.google.com/store/apps/details?id=com.fidesmo.sec.android&hl=es)
- [Unidirectional flow to tame the state](http://monday8am.com/blog/2018/03/30/unidirectional.html)
- [Fastlane and Git working together](http://monday8am.com/blog/2018/02/15/fastlane.html)
