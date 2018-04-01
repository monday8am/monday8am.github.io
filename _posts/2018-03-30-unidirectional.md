---
layout: post
title:  "Unidirectional flow to tame the state"
date:   2018-03-30 09:30:01 +0200
categories: blog
---

[Unidirectional architectures](https://staltz.com/unidirectional-user-interface-architectures.html) are here to stay. Web development is full of successful cases based on [Flux](https://facebook.github.io/flux/), [Redux](https://redux.js.org/), [Elm](http://elm-lang.org/), [React](https://reactjs.org/) among others and some of them have been ported to mobile dev with really nice results. But, are these ported frameworks enough mature to be used into a production environment? Is it worth it?

At Fidesmo we think **it is**, but there are some considerations about our app that make the decision easier. Our app is a kind of bridge between [Fidesmo devices](https://developer.fidesmo.com/javacard) and a [whole remote platform](https://developer.fidesmo.com/architecture) to interact with them. So, for our app:

- There are many (and well defined) states triggered by the three different inputs: *the user*, *the network* and *the device*. Most of the time we could describe our app behaviour as a finite state machine.

- It is developed using Kotlin and Swift and both languages are suitable for a functional programming approach.

- It doesn't have many screens so the whole application state is not too nested.

### Framework

At the time to select a framework, we opted for [ReSwift](https://github.com/ReSwift/ReSwift) as the most mature option for iOS and [ReKotlin](https://github.com/ReKotlin/ReKotlin) for Android, in order to use almost the same definitions in both platforms. Both (ReSwift and ReKotlin) are implementations of Redux.

Following the Redux patterns our application architecture looks like this:

![Screenshot]({{ "/assets/img/graphic_unidirectional_flow.png" | absolute_url }})

Besides the standard Redux components (_actions_, _state_, _reducers_) we want to highlight the role of **middlewares** inside the architecture. They catch actions and according to the state at this moment, perform async operations and generate new actions. For example:

{% gist 289446f2ed0a29822f8ba8be3639239a %}

### State definition

Both Kotlin and Swift have structures to define algebraic data types (like a real funcional language) and therefore to describe complex states as we need. In case of Swift, we can use its fantastic **enum** implementation and using Kotlin, we can achieve almost the same using **sealed classes**:

The _delivery_ state of our app described in Swift

{% gist 7eda7e7873147f6315473105d0345557 %}

and the same state translated to Kotlin

{% gist 98aaf7290f2af3c0b1dc410d6257bb80 %}

### Conclusions

Although Kotlin and Swift are not as similar as we thought at first, implementing the same patterns with them is a little step forward the dream of "write once, run anywhere".

The current codebase is really enjoyable, besides the advantages of the functional approach and the unidirectional data flow: simplicity, predictability, ridiculous easy testing.

You should give a try for sure! :)

#### Links

- [Algebraic data types on Kotlin](https://medium.com/car2godevs/kotlin-adt-74472319962a)
- [Modelling state in Swift](https://www.swiftbysundell.com/posts/modelling-state-in-swift)
- [Middleware introduction](https://medium.com/@jacobp100/you-arent-using-redux-middleware-enough-94ffe991e6)
- [Unidirectional data flow on Android](https://proandroiddev.com/unidirectional-data-flow-on-android-the-blog-post-part-1-cadcf88c72f5)
- [Unidirectional data flow on iOS](https://academy.realm.io/posts/benji-encz-unidirectional-data-flow-swift/)
