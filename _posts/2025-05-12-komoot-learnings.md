---
layout: post
title: "Komoot Learnings: Cutting the Gordian Knot"
date: 2025-05-12
categories: blog
---

![Screenshot](/assets/img/alexander-gordian-knot.webp)

*Legend tells of Alexander the Great encountering the **Gordian Knot**, an impossibly complex tangle that had defeated all who attempted to unravel it. While others tried to patiently trace each strand, Alexander drew his sword and cut straight through, solving in seconds what had seemed unsolvable for years.*

*In software engineering, we often face our own Gordian Knots — codebases so tightly coupled that untangling them step by step feels impossible. But sometimes, the most effective solution is to make a clean cut, isolate what you need, and start fresh.*

**That's the story of how we modularized Komoot's Android codebase.**

## The context

My last company was the leading outdoor navigation app in Europe. When I joined, the Android codebase had accumulated 17 years of legacy: navigation based on activities, no dependency injection, a monolithic module structure, and only isolated attempts at adopting ViewModels or Compose. The result was long compilation times, painful debugging, and fragile release cycles — slowing feature development and making onboarding demotivating.

A few weeks later, I was assigned to revamp the route search tool. **I proposed building it entirely from scratch using Jetpack Compose and a modern stack**, using it as a **trampoline for broader architectural reform**. If it succeeded, it could:

* Prove that feature development using modern tools was viable, faster and perform better, even within the legacy context.
* Serve as a reference point to inspire modernization across the codebase.
* Create a healthier and more motivating work environment — something that, for me, was personally essential to continue on the project.

I led both the feature development and the broader modernization effort. But the challenge was significant: how could we both deliver a working feature and plant the seeds for large-scale change?

## The plan

**How do I trigger a global scale refactoring and develop a feature at the same time?**

The goal was to define a minimal and optimized list of actions that permit us to create a favorable context for a fast Compose feature development, and at the same time, introduce improvements with global impact. Those actions had to be executed one by one to reduce risks and if possible, have measurable outputs. Based on the critical needs on both app and feature levels, I defined the following list to be executed in cascade:

1. **Introduce a dependency injection framework** (Hilt) that improves code quality at a global scale and permits moving dependencies to different modules, unblocking a module split action.

2. **Extract a feature module**: At a global scale, solves the problem of a main module constantly growing. For feature development, reduces the compilation time drastically improving the use of Compose previews. Besides, isolates the new code and permits feature sample apps, accelerating the development even more.

3. **Create UI in Compose**: At a global scale, add an incipient design system module with core UI components. Besides, introduces a change in the data management paradigm from an imperative to a reactive approach, causing a necessary update in the upper layers. At the feature level, speed up the development.

4. **Rewrite the upper layers**: The existing models and data sources were written in Java and translated to Kotlin, making them hard to reuse. It motivated an encapsulation of the network layer with mappers and interfaces and rewrote the data layer using coroutine flows.

## Breaking the monolith

Our biggest obstacle was the `Application` class — a God object acting as a service locator for virtually every system in the app. Everything depended on it, and it depended on the monolith. A circular dependency problem that made any module extraction seem impossible.

Our solution was surgical but effective:

![Screenshot](/assets/img/app-komoot-split.webp)

We extracted `KomootApplication` as an interface containing only the essential services that the legacy code needed to access, and placed it in a new shared module. Then we created a new `:app-komoot` module containing only the Android application entry point and the implementation of that interface. The massive `:komoot` module remained as "legacy" but now received the application context through the interface rather than directly accessing the `Application` class.

With that structure in place, we gradually introduced Hilt. This was crucial for allowing new modules to declare their dependencies without reaching up through the God object pattern.

The `:feat-atlas` module was created at the same level as `:komoot`, not nested within it. *This was psychologically important* — it meant that new features would be first-class citizens, not subordinate to the legacy code.

Today, **the codebase has over 70 modules**, including **30 feature modules**. Jetpack architecture (ViewModel, Compose, DI) is now standard across the team. *Most importantly, the team defined a shared "North Star" architecture and has consistently moved toward it*.

## Scope and delivery

The project was scoped with a dual objective: deliver the feature and lay the groundwork for long-term improvements.

To do so, I negotiated an initial three-week shaping phase—a rare opportunity to build infrastructure and isolate the new module before feature work began. And fortunately this paid off, development proceeded faster than expected, driven by:

- High team motivation.
- A modern stack.
- Significantly shorter build times due to modularization.
- A dedicated app to run the feature

For the broader architecture shift, we viewed it as a multi-phase journey. The feature served as a proof of concept. Full adoption would take months or even years, but we had a foundation and direction.

## How did we de-risk the process?

* **Team adoption:** Ran workshops, presentations, and Q&A sessions to reduce resistance, foster understanding, and increase ownership.
* **Controlled refactoring:** Each step was executed sequentially to limit blast radius and quickly identify regressions.
* **Feature stability:** Feature flags and ViewModel unit tests ensured confidence during rollout.

## The numbers

We didn't want to rely on subjective feelings about whether modularization was working. Using [Gradle Profiler](https://github.com/gradle/gradle-profiler), we established baseline measurements on the master branch before any modularization work. We measured ABI changes, non-ABI changes, Compose changes, and clean builds — then ran the same benchmarks on the branch containing our modularization work.

Before and after Gradle scans revealed dramatic improvements:

![Screenshot](/assets/img/gradle-scan-results.webp)

**Build cache effectiveness** improved from **16.8%** to **49%** cached tasks. Parallelization became significantly more effective, with much better distribution of work across CPU cores.

![Screenshot](/assets/img/gradle-profiler-results.webp)

**Build times for `:feat-atlas` were 7x faster than for `:komoot` across all scenarios:**

* **ABI changes:** Atlas ~8.4s vs Komoot ~49s
* **Non-ABI changes:** Atlas ~8.3s vs Komoot ~47s
* **Compose changes:** Atlas ~10.5s vs Komoot ~51.8s

All tests in the Atlas module completed in under 9 seconds, including Compose changes. And the Atlas module already contained complex functionality — these weren't artificially simple benchmarks.

### A note on honesty

The performance gains were real, but they were also largely *expected*. 

The harder question was: *how do you keep the modules healthy over time?* How do you prevent new monoliths from forming, enforce boundaries, catch dependency creep before it becomes a problem?

**We didn't fully solve that.** We used Gradle profiler traces to demonstrate the gains, but a long-term module health strategy — automated dependency rules, module size budgets, regular audits — that's the work that comes after the dramatic chart, and we didn't have time to implement it. It remains an open problem.

## Learnings

I learned that meaningful technical change doesn't always come from building the most complex system — it can also come from the courage to improve what's broken.

In this project, the greatest challenge was not technical, but cultural: convincing a team to let go of familiar inefficiencies and embrace something new. This required constant communication, shared ownership, and letting go of control. I had to facilitate, not just lead. It reminded me that engineering leadership is about creating momentum, not just writing clean code. And sometimes, it's about fighting for the space to do things right, even when it's hard and resistance feels overwhelming.

If I were to do this project again, I'd follow the same approach, though I'd focus even more on facilitation and less on debate :)

## Links

* [Android at Scale — Droidcon](https://www.droidcon.com/2019/11/15/android-at-scale-square/)
* [A complete journey of Android modularization](https://www.droidcon.com/2022/08/01/a-complete-journey-of-android-modularisation/)
* [Forging the path from monolith to multimodule app](https://www.droidcon.com/2022/08/01/forging-the-path-from-monolith-to-multi-module-app/)
* [Improve Android productivity](https://www.droidcon.com/2022/08/03/5-ways-to-improve-your-android-build-productivity/)
* [Speeding up your Android Gradle builds (Google I/O '17)](https://www.youtube.com/watch?v=7ll-rkLCtyk&t=1821s&ab_channel=AndroidDevelopers)
* [Farewell Komoot](https://www.monday8am.com/blog/farewell-komoot/)
