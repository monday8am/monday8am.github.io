---
layout: post
title:  "Komoot learnings"
date:   2025-05-12 09:30:00 +0100
categories: blog
---

## Summary

My last company was the leading outdoor navigation app in Europe. When I joined, the Android codebase had accumulated 17 years of legacy, with navigation based on activities, no dependency injection, a monolithic module structure, and only isolated attempts at adopting ViewModels or Compose. The codebase was so large that modernizing it would have been too costly in terms of time and effort. The existing team felt a mix of resignation and comfort with the inefficient, but very familiar, code

The technical issues directly affected day-to-day work: long compilation times, painful debugging, and fragile release cycles. This not only slowed feature development but also made onboarding new engineers difficult and demotivating.

A few weeks later, I was assigned to revamp the route search tool. **I proposed building it entirely from scratch using Jetpack Compose and a modern stack**, using it as a **trampoline for broader architectural reform**. 
If it succeeded, it could:

* 	Prove that feature development using modern tools was viable, faster and perform better, even within the legacy context.
* 	Serve as a reference point to inspire modernization across the codebase.
* 	Create a healthier and more motivating work environment—something that, for me, was personally essential to continue on the project.

I led both the feature development and the broader modernization effort. But the challenge was significant: how could we both deliver a working feature and plant the seeds for large-scale change?

Key decisions revolved around:

* 	Balancing delivery with strategic refactoring.
* 	Deciding what to build from scratch vs. reuse.
* 	Convincing the team to invest more today for a better future.

After a lot of hard work finishing and polishing the feature, it performed exceptionally well. It was promoted to the main tab (Routes) and became the default screen for new users. Its architecture was adopted as a blueprint for future features.

Today, **the codebase has over 70 modules**, including **30 feature modules**. Jetpack architecture (ViewModel, Compose, DI) is now standard across the team. *Most importantly, the team defined a shared “North Star” architecture and has consistently moved toward it*.


## The Plan

**How do I trigger a global scale refactoring and develop a feature at the same time?** 

The goal was to define a minimal and optimized list of actions that permit us to create a favorable context for a fast Compose feature development, and at the same time, introduce improvements with global impact. Those actions had to be executed one by one to reduce risks and if possible, have measurable outputs. Based on the critical needs on both app and feature levels, I defined the following list to be executed in cascade:

1. **Introduce a dependency injection framework** (Hilt) that improves code quality at a global scale and permits moving dependencies to different modules, unblocking a module split action.

1. **Extract a feature module**: At a global scale, solves the problem of a main module constantly growing. For feature development, reduces the compilation time drastically improving the use of Compose previews. Besides, isolates the new code and permits feature sample apps, accelerating the development even more. 

1. **Create UI in Compose**: At a global scale, add an incipient design system module with core UI components. Besides, introduces a change in the data management paradigm from an imperative to a reactive approach, causing a necessary update in the upper layers. At the feature level, speed up the development.

1. **Rewrite the upper layers**: The existing models and data sources were written in Java and translated to Kotlin, making them hard to reuse. It motivated an encapsulation of the network layer with mappers and interfaces and rewrote the data layer using coroutine flows.

## How Did We De-risk the Process?

- **Team adoption:** Ran workshops, presentations, and Q\&A sessions to reduce resistance, foster understanding, and increase ownership
- **Controlled refactoring:** Each step was executed sequentially to limit blast radius and quickly identify regressions
- **Instrumentation:** The modularization progress was profiled and measured
- **Feature stability:** Feature flags and ViewModel unit tests ensured confidence during rollout


## Scope and delivery

The project was scoped with a dual objective: deliver the feature and lay the groundwork for long-term improvements. 

To do so, I negotiated an initial three-week shaping phase—a rare opportunity to build infrastructure and isolate the new module before feature work began. And fortunately this paid off: development proceeded faster than expected, driven by:
	•	High team motivation.
	•	A modern stack.
	•	Significantly shorter build times due to modularization.
    •	A dedicated app to run the feature

For the broader architecture shift, we viewed it as a multi-phase journey. The feature served as a proof of concept. Full adoption would take months or even years, but we had a foundation and direction.

## Learnings

I learned that meaningful technical change doesn’t always come from building the most complex system—it can also come from the courage to improve what’s broken. 

In this project, the greatest challenge was not technical, but cultural: convincing a team to let go of familiar inefficiencies and embrace something new. This required constant communication, shared ownership, and letting go of control. I had to facilitate, not just lead. It reminded me that engineering leadership is about creating momentum, not just writing clean code. And sometimes, it’s about fighting for the space to do things right, even when it’s hard and resistance feels overwhelming.

If I were to do this project again, I’d follow the same approach, though I’d focus even more on facilitation and less on debate :)
