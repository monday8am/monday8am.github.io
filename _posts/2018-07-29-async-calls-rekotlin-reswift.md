---
layout: post
title:  "Call async code using ReKotlin/ReSwift"
date:   2018-07-29 19:37:00 +0100
categories: blog
---

One of the most chocking ideas (at least for me) after adopt a _Redux like_ pattern is that everything works totally synchronous. The actions are processed one after another and the view is updated responding to synchronous modifications of the state. Besides the actions are processes in the main thread and the predictability is fantastic but: _How to deal with asynchronous signals or remote API calls?_

### The two paths of the "Async way"

There are two main different approaches to deal with async calls: _action creators_ and _middleware_.

Both of them share the same functionality behind and it can be summarized in 3 main points:

1. **There is a trigger to make them work:** Of course, something needs to happen in order to call an async method but the nature of this trigger diffiers from middleware to action creators. Action creators are called directly and then the async code inside them is executed. Middleware async code is called only when some kind of action is detected.

2. **The existing state is used to decide if do something:** Before call the async code, a copy of the whole state is needed in order to check if the call is really needed or not. Both action creators and middleware injects a copy of the current state inside their implementations.

3. **An action is dispatched:** We should dispatch actions to notify the _external world_ how the async call is going. For example, when the async method finish we dispatch the result (success or error) inside an action. Besides, we can dispatch a _loading_ action before start the asyn call.

translated to seudo code, these three points looks like:

{% gist 52c76962e0281b25a60b081f5004aad5 %}

both _ReKotlin_ and _ReSwift_ include an easy way to define middleware and action creators. Let's take a look to both of them:

#### Action creators

As we see, action creators are methods with the signature:

`(state: State, store: StoreType) -> Action?`

The state parameter permits to check the current state information and store will be used to dispatch the actions we need:

{% gist bcd3d13680185bd4d21b701402fbf2f2 %}

#### Middleware

The middleware is defined by a very scary signature. Considering the a dispatch function like: `(Action) -> Unit`, a middleware can be defined as:

`(DispatchFunction, () -> State?) -> (DispatchFunction) -> DispatchFunction`

Weird, but let's check the code to see that they are really simply to use:

{% gist 66b8b9a9dbe1c4ec9981e5ba11df8511 %}

One interesting point is that you can define a list of middlewares and they will acts like an assembly line. For that reason, there are two kinds of dispatch functions inside them:

- _dispatch_: The action will be inserted at the start of the middleware chain.
- _next_: The action dispatched will continue to the next middleware in the chain or to the reducers.

### Conclusions

As we have seen, async calls can be implemented easily in ReSwift and ReKoltin app. Personally I prefer middleware because the async calls can be grouped together according to the type of API. For example, _DeviceMiddleware_ to deal with bluetooth async calls, _NetworkMiddleware_ for the remote API and _LoggingMiddleware_ to register info about all the actions.

#### Links

- [ReSwift docs](https://reswift.github.io/ReSwift/master/getting-started-guide.html)
- [You Aren’t Using Redux Middleware Enough](https://medium.com/@jacobp100/you-arent-using-redux-middleware-enough-94ffe991e6)
- [Putting ReSwift Actions on the Undo Stack Using Middleware](https://christiantietze.de/posts/2016/08/reswift-undo-middleware/)
