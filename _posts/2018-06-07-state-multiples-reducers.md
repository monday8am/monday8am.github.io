---
layout: post
title:  "State and multiples reducers"
date:   2018-07-06 17:25:00 +0200
categories: blog
---

If you are working with Redux, the _state definition_ is a key task in the design of your application. Once the state is defined, the reducers are the next step, setting how the state will change according to the inputs received. If your state is too complex, is better to group the logic in two or more reducers and then use something like the following code:

```val newState = combineReducers(::reducerA, ::reducerB)(action, oldState)```

In this way, the method `combineReducers` will accept as many reducers needed, connecting the output of one reducer to the input of the next one in the list. All of them will modify the state in a functional and testable way and the logic will be separated in smaller modules.

Let's define the [Reducer](https://github.com/ReKotlin/ReKotlin/blob/master/src/main/kotlin/org/rekotlin/Reducer.kt) interface and the [combineReducers](https://redux.js.org/api/combinereducers) method for Kotlin:

{% gist 9827bc69b51fe88b8726c9d05265492a %}

### Conclusions

If a concept is exported from the web and JS world (Redux in this case) then many solutions will be found in the same place (original Redux documentation). Besides, Kotlin and Swift give enough flexibility to reproduce any solution needed.

#### Links

- [Method combineReducers in Redux](https://redux.js.org/api/combinereducers)
