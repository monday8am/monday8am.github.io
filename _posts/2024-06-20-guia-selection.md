---
layout: post
title:  "Choosing a Compose Navigation Library (Draft)"
date:   2024-06-20 10:30:00 +0100
categories: blog
mermaid: true
---

At some point during the Komoot Android modernisation, the balance tipped. We'd been adding Compose screens one by one — a new feature here, a rewritten flow there — but with Project Arrow, entire areas of the app were becoming full Compose. The discovery tab, the map, the planner. Not isolated screens embedded in activities, but interconnected Compose flows with their own navigation stacks, bottom sheets, and shared state.

We needed a navigation library. Not eventually — now.

The problem was that Google's Jetpack Navigation for Compose, at the time, couldn't do what we needed. And several of the things we needed weren't exotic requirements — they were basic expectations for a production app with complex UI.

## What we needed

The requirements came from the app as it existed, not from an ideal architecture document. Each one had a concrete reason behind it:

**Complex parcelable data in navigation parameters.** This was the dealbreaker with Jetpack Navigation Compose. We needed to pass rich data objects between screens — route configurations, filter states, map initialisation parameters. Jetpack Navigation Compose at the time worked with string-based routes and primitive argument types. Passing a `PlannerInit` data class with nested objects meant serialising to JSON strings and deserialising on the other side, which was fragile and lost type safety. We needed keys that were just `Parcelable` data classes, carried directly on the backstack.

**Multiple container types.** Screens, bottom sheets, and dialogs had to be first-class navigation destinations — not afterthoughts bolted on with separate APIs. In the Atlas flow, a user might navigate from a full-screen map to a bottom sheet showing tour details to a dialog confirming a download, all within the same navigation graph.

**Observable backstack.** The map tab needed to observe what was on the navigation stack so it could adjust the map camera, show or hide UI elements, and react to navigation changes. A black-box backstack wasn't going to work.

**Nested navigation graphs.** The bottom navigation had two tabs (Discover and Map), each with its own independent navigation stack. Screens within a tab needed their own sub-navigation without interfering with the other tab's state.

**Custom lifecycle and state persistence.** Each screen needed its own `ViewModelStoreOwner` so Hilt-injected ViewModels were scoped correctly. Screen state had to survive process death through `Parcelable` keys and `rememberSaveable`.

**Simple enough to fork.** This was a pragmatic requirement. We knew we'd need to modify internals — adding container types, adjusting lifecycle behaviour, integrating with our existing architecture. A library with 50,000 lines of multiplatform code wasn't going to work, no matter how feature-complete.

## The landscape

The Android community had produced several alternatives to Jetpack Navigation. I evaluated six of them against our requirements:

**[Appyx](https://github.com/bumble-tech/appyx)** brought interesting ideas about navigation as something more complex than a stack — tree-based hierarchies, custom transition models. Well documented, with conference talks explaining the concept. But it started as an internal Bumble tool, and the abstraction was too generic for what we needed. Hard to integrate, hard to customise.

**[Decompose](https://github.com/arkivanov/Decompose)** shared DNA with Appyx (same original creator) and pushed even further into framework-agnostic, multiplatform territory. Innovative, but the concepts were too far from what the team expected. No type safety for navigation arguments. Integration would have required rethinking how we structured everything.

**[Compose Destinations](https://github.com/raamcosta/compose-destinations)** took a different approach — annotations on composable functions, processed by KSP to generate navigation code. Low learning curve since it builds on Google's own library. But the annotation processing added compile time, and more importantly, the navigation code was tightly coupled to the UI. We wanted navigation definitions separated from screen implementations.

**[Voyager](https://github.com/adrielcafe/voyager)** was the most feature-complete option. Popular, well documented, good Android platform integration, Parcelize support. It checked most boxes. But it was a multiplatform library with a large codebase. Forking it and maintaining our own version would have been painful. It also lacked support for custom stack types (like wizard flows).

**[Compose Navigation Reimagined](https://github.com/olshevski/compose-navigation-reimagined)** was clean and close to Google's approach but simpler. Short source code, easy to understand. Good ViewModel scoping through navigation graph scopes. But no custom container types (no built-in bottom sheet or dialog navigation), no multi-module example, and the project looked stale at the time.

**[Guia](https://github.com/roudikk/guia)** was the smallest of the bunch. Simple, flexible, well documented for its size. Three built-in container types (screen, bottom sheet, dialog). Short source code that was easy to read through in an afternoon. Different stack types supported. The trade-off: small community, not many users, not the best ViewModel scoping story out of the box.

## Why Guia won

Guia and Reimagined were the two finalists. Both had codebases short enough to fork. Both used typed keys. Both treated the backstack as state.

What tipped it was the container model. Guia treated screen, bottom sheet, and dialog as three types of `NavigationNode` — the same key could be displayed in any container depending on the registration:

```kotlin
fun NavigatorConfigBuilder.detailsNavigation(screenWidth: Int) {
    if (screenWidth <= 600) {
        dialog<DynamicDetailsKey> { DetailsContent(item = it.item) }
    } else {
        bottomSheet<DynamicDetailsKey> { DetailsContent(item = it.item) }
    }

    screen<DetailsKey> { DetailsScaffold(item = it.item) }
}
```

Same key, different presentation based on context. That flexibility matched how our UI actually worked — the Atlas flow needed bottom sheets on phones and dialogs on tablets for the same content.

The other deciding factor was the navigation key model. Guia's `NavigationKey` was a plain `Parcelable` interface. Our keys were data classes carrying real domain objects:

```kotlin
@Parcelize
data class AtlasKey(
    val init: AtlasInitContent = AtlasInitContent.Default
) : NavigationKey

@Parcelize
data class PlannerKey(val init: PlannerInit) : NavigationKey

@Parcelize
data class SearchTypeSelector(
    val sport: FavoriteSportTopic,
    val latitude: Double?,
    val longitude: Double?
) : NavigationKey
```

No string routes. No argument bundles. No serialisation gymnastics. The key *was* the data, and it lived on the backstack as a first-class citizen. Process death? The key is `Parcelable` — it restores automatically.

Reimagined could do most of this too, but its container model was more limited and extending it would have required deeper changes to the library's internals.

## What we changed

We forked Guia and adapted it over time to fit our production needs. The core concepts stayed intact — `NavigationKey`, `NavigationNode`, `Navigator` with a state-backed backstack — but several areas needed work.

<!-- TODO: Add code examples of customisations once ready. Areas to cover:
- ViewModel scoping changes (ViewModelStoreOwner per backstack entry)
- LifecycleManager additions
- ResultManager for screen-to-screen data passing
- NavHost for multi-stack tab navigation
- Integration with Hilt
- Any transition system changes
-->

*[This section will be expanded with implementation details.]*

The key principle behind the modifications: keep the library's surface area small and the concepts simple, but make the internals production-grade. We wanted something the whole team could understand after reading the source once, not a framework that required a manual.

## The validation

After I left the project, Google released [Jetpack Navigation 3](https://developer.android.com/guide/navigation/navigation-3). Reading through the API was satisfying — not because it matched our code line by line, but because the requirements we identified as essential turned out to be the same ones Google addressed in their rewrite.

Nav3 uses typed keys (`NavKey`). The backstack is developer-owned Compose state. Navigation is adding and removing items from a list. Multiple backstacks are supported natively. The approach Google arrived at is structurally the same as what Guia (and Reimagined, and Voyager) had been doing — because the Compose runtime pushes everyone toward the same shape.

The practical consequence: migrating from our Guia fork to Nav3 would be a mechanical refactoring. The keys translate directly. The backstack model is equivalent. The container registration is similar. The bet we made — that key-based, state-driven navigation was the right abstraction — held up.

Choosing a small, forkable library over a popular but large one meant we could move fast, customise freely, and stay close to the concepts that the platform was converging toward anyway. Sometimes the right choice isn't the most popular library — it's the simplest one that covers your requirements and gets out of your way.
