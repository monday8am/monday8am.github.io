---
layout: post
title:  "Rx-ify your APIs for better understanding!"
date:   2018-11-18 13:30:00 +0100
categories: blog
---

### Async is everywhere

We could define -in a simplistic way- the *async code* as the code that, after calling it, returns an answer that is not inmediately available and most of the time is executed in a background thread. For example, to connect to a BLE device or to download an image from server.

Nowadays *async code* is everywhere and tech companies and opensource communities have made efforts to make it more legible and easier to deal with. The solutions varies in name and format depending on the languages and the platforms: we can find simple [callbacks](https://en.wikipedia.org/wiki/Callback_(computer_programming)), [futures or promises](https://en.wikipedia.org/wiki/Futures_and_promises), syntactic sugar like [async-await](https://javascript.info/async-await) or compiler tricks like [coroutines](https://kotlinlang.org/docs/reference/coroutines/coroutines-guide.html).

### Reactive extensions

Together with the solutions mentioned above, we have the [Reactive Extensions](http://reactivex.io/) framework. The Rx creates a channel of signals with all the future async inputs (_Observable_) that can be consumed by anyone who subscribe to it (_Observer_).

Besides, the signals inside the channel can be altered using wellknown functional operators like _map_, _flatMap_ or _reduce_ and the channels can be connected with other channels to create a _Rx pipeline_. For example, you can build a pipeline that waits for a user input that triggers a remote call to a rest API followed by a complex calculation over the result, all of it in few lines of code!

### Applying the Rx style

If you're a MacOs or iOS developer is very common to find system APIs using callbacks: [], [], []. Although they could break our app style introducing another way to deal with asynchronism, move them to the bright side is extremely easy in two steps:

- Create a utility class that mimic the API interface.
- Add an internal delegate to intercep the callacks and transforms it into observables.

Let's create it for the `CBCentralManager` API:




 
### Conclusions

#### Links

- 