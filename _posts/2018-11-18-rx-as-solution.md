---
layout: post
title:  "Rx-ify the system APIs for better use and understanding!"
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

If you're a MacOs or iOS developer is very common to find system APIs that needs delegates to deal with the async calls: [CBCentralManager](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager), [CBPeripheral](https://developer.apple.com/documentation/corebluetooth/cbperipheral) or [NFCReaderSession](https://developer.apple.com/documentation/corenfc/nfcreadersession) for example. Although they could break our app style introducing another way to deal with asynchronism, move them to the bright side can be accomplished in two easy steps:

- Create a utility class that mimic the API interface.
- Add the delegate as internal class to intercep the answers and transforms it into observables.

Let's create it for the `CBCentralManager` API:

{% gist adc0880095dd81a6f66af91ee24bf5f6 %}

Once the utility class is ready, we can use the API in a more Rx way:

{% gist 2aebdff7216b09498771393eb50ba664 %}

### Conclusions

Understanding and predictability is very important in the context of an application development and in many cases is better to introduce a bit more code but with a high impact in the whole codebase readability. 

If you are already using Rx libs, maybe the introduction of a Rx layer on top of a system API takes some effort but at the moment of use it together with other APIs or with UI code they will fit lot better, increasing productivity and flexibility.

#### Links

- [Reactive Extension portal](http://reactivex.io/)
- [RxSwift](https://github.com/ReactiveX/RxSwift) and [RxKotlin](https://github.com/ReactiveX/RxKotlin)
- [RxBluetoothKit](https://github.com/Polidea/RxBluetoothKit): example of a Rx layer on top of the [CoreBluetooth](https://developer.apple.com/documentation/corebluetooth) API