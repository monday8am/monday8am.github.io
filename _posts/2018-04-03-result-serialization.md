---
layout: post
title:  "JSON serialization and generics in Kotlin."
date:   2018-04-03 20:02:00 +0200
categories: blog
---

### Preface

Sometimes at the moment of create an app, you have to introduce some complexity in order to keep the global code base simpler.

Applying this concept to a bicicle I think in an [internal-gear hub](https://en.wikipedia.org/wiki/Hub_gear) part: the interaction with it is very easy (only one spoke) and makes the life easier if you stop-and-go when commuting in a big city. On the other hand, it is a complex and expensive component.

Curiosly, the same example can be found inside [Fidesmo](http://fidesmo.com/) app codebase.

### The easy interaction

We use a version of the _Either<_**A**_,_**B**_>_ pattern redefined as _Result<_**T**_,_**E**_>_ . Popular in languages like Scala, Elm or Rust, it is fantastic because it wraps a value of type **T** inside a "request context", where **E** is the error obtained if the request fails. Although you can find [many implementations](https://github.com/michaelbull/kotlin-result) around, let's type a simple one:

{% gist 02cc2ae467845e1ef8ecacfaa56bcca9 %}

### The complex part

As we are using a [Redux like architecture](/blog/2018/03/30/unidirectional.html) inside Fidesmo app, the _Result<_**A**_,_**B**_>_ is part of our state and it must be **serialized**!! 😱

How to serialize a sealed class with generics in Kotlin? In theory, we should add a custom adapter to the Google Gson lib that let's us customize the JSON (de)codification. This adapter could have the follow structure:

{% gist 8370a8f1f3135c32d6c358b1065770c2 %}

#### Result<T, E> detection

In order to pass the previous serializer to the Gson implementation, we need something that detects the whole hierarchy of our sealed class: _Result.Loading_, _Result.Ok<_**T**_>_ or _Result.Err<_**E**_>_. 

And _voilá_, The `registerTypeHierarchyAdapter` comes to help us:

{% gist fa98275257ccb7b3dee5bb92e408af7e %}

#### Result<T, E> serialization

About codification, we have predefined that we'll save only the `Ok` result and discard the other states. A pattern matching can help us to execute the correct action for each case:

{% gist befe53b902ededc1555df24669e6023a %}

#### Result<T, E> deserialization

Ok, fasten your seat-belts:

{% gist 950565efe888dc8f53e5f34f254b1d23 %}

### Conclusions

Make an application (or software in general) is matter of decisions, most of them are not trancendentals but seen together as a whole can lead your code to a land of pleasure full of unicorns or to the terrible hell. About this case, the introduction of a complex module in benefict of the overall simplicity is a good one :)

#### Links

- [Result monad implementation](https://github.com/michaelbull/kotlin-result)
- [Result monad implementation version 2](https://github.com/danneu/kotlin-result)
- [Full Fidesmo monad pattern](https://gist.github.com/monday8am/c934490a42f87ea33f01083790295304)
- [Full Fidesmo Json ResultSerializer](https://gist.github.com/monday8am/bce54f4852ae9ba70a3c28413b6e46d0)
- [Serialization of generics in Java](https://marcin-chwedczuk.github.io/generics-in-java)
