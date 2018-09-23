---
layout: post
title:  "Call async code using ReKotlin/ReSwift"
date:   2018-07-29 19:37:00 +0100
categories: blog
---

One of the most chocking ideas (at least for me) when a Redux like pattern is adopted is that it works totally synchronos. The actions are processed one after another and the view is updated responding to syncronos modifications of the state.

Considering that actions are dispatched and processes in the main thread the predictability is fantastic but, how to deal with asynchronous signals? How to manage remote API calls or BLE connections?

### The two paths of the "Async way"

There are two main different approachs to deal with asyn calls: _action creators_ and _middlewares_ . Both of them share the same functioning behind and it can be summarized en 3 main postulates:

**There is a trigger to make them work:** Of course, something needs to happen in order to call an async method but the nature of this trigger diffiers from middlewares to action creators. Action creators are called directly and then the async code inside them is executed. Middleware async code is called only when some kind of action is detected.

**The existing state is checked to do something (or not):** Before call the async code, a copy of the whole state is needed in order to check if the call is really needed or not. Both action creators and middlewares injects an atomic copy of the state inside their implementations.

**An action is dispatched (or not):** When the async method finish we need to notify to the state that the result (or an error) was obtained. The way to achieve it is dispatching and action with the result. Besides, we can dispatch a Loading action before start the asyn call.

translated to seudo code, it could looks like:

{% gist 52c76962e0281b25a60b081f5004aad5 %}

Both ReKotlin and ReSwift include an easy way to define middlewares and action creators. Let's take a look to both of them: 

### Action creators definition and use

As we see, action creators are method with the signature _(store: state)_. The state parameter permits to check the current state information and store will be used to dispatch the actions we need:

{% gist bcd3d13680185bd4d21b701402fbf2f2 %}
(incluir como usarlo en el codigo)

### Middleware definition and use

{% gist 66b8b9a9dbe1c4ec9981e5ba11df8511 %}

Personally I prefer middlewares because the related async calls can be grouped according to the type of API they use. For example, BleMiddleware to deal with Ble async calls, NetworkMiddleware for the remote API and LoggingMiddleware to get info about all the actions.

### Conclusions

#### Links
