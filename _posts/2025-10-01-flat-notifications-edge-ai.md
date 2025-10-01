---
layout: post
title:  "From flat Notifications to Edge AI"
date:   2025-10-01 14:23:00 +0100
categories: blog
---

### _A Yazio-Inspired Experiment with an agentic framework, local model inference and a small language model_

### Introduction

For the last few weeks I have been testing the [**Yazio**](https://www.yazio.com/) app, a calorie counter. While using it, I noticed that the notifications — though helpful — were sometimes similar and easy to ignore. This observation sparked a question:

_Could these reminders be generated dynamically, on the device, based on my context, and sound more natural or timely?_

That question led me into an experiment that combined JetBrains Koog as agentic framework and Google MediaPipe Inference API + Gemma LLM for Android edge AI.

The result? A [prototype](https://github.com/monday8am/koogagent) for smarter, more context-aware messages — powered by a local model, and a fantastic starting point for learning and experimenting with generative AI.

### The problem: Flat notifications

Notification messages are a crucial tool for many apps, and in multiple cases, the main entry point to them. In the case of Yazio, they do a good job reminding users to log meals or water intake.

But after a while, they can start to feel repetitive or disconnected from what’s actually happening in our day. Maybe they arrive when we’re in the gym, or suggest drinking water instead of a cup of tea on a cold day. It’s a natural limitation of _static, predefined messages_. They don’t adjust to:

- What you’ve already logged
- What time it is
- Your habits or preferences
- Your current activity or mood

In other words, _they lack context_.

### **Why Context Matters**

Behavioural psychology suggests that _timing, tone, and context_ deeply affect how people respond to messages. Our moms know exactly if we’re hungry, and our favourite dishes for each situation. That’s because they have years and years of context about us (and we trust them! but that’s another topic ☺).

A notification like:

> Time for a snack?

vs.

> Nice pace today. Since lunch is logged, a quick summer bite — gazpacho or yogurt with peach — will keep you moving 💪”

It could feel more personal and relevant — not by guessing, but by _responding to context_. Not quite like your mom, but better than a static reminder.

Hardcoding these messages isn’t scalable, but solving problems of generated language and _nondeterministic outputs_ is exactly where language models shine.

### Using Koog to Emulate an Agent

[JetBrains Koog](https://github.com/JetBrains/koog) is an [agentic framework](https://www.ibm.com/think/insights/top-ai-agent-frameworks) designed to build _reasoning agents_ around language models. It’s created in Kotlin and instead of making a raw prompt request, Koog helps implement the following chain:

![Screenshot]({{ "/assets/img/koog-agentic-flow.png" | absolute_url }})

Koog provides connectors for accessing to data using [Model Context Protocol](https://www.anthropic.com/news/model-context-protocol) (MCP) and common large language model APIs, orchestrating tools and decisions around a model. In this experiment, Koog triggers an agent on notification receipt, assembles user and device context, generates a response, and applies safety post‑processing

At the time of writing, Koog _doesn’t natively target small, on‑device models_ — but it can still orchestrate local inference as part of a hybrid setup.

### The Edge AI choice

When designing this feature, I believed a _hybrid architecture_ was the best solution: small language models ([SLMs](https://huggingface.co/blog/jjokah/small-language-model)) for the default local workflow, with a large model in the cloud as a fallback. It is both a strategic and a bold decision.

- Strategic, because the future of agentic AI is moving toward SLM-first systems: faster, cheaper, and easier to align for repetitive subtasks, [as NVIDIA’s research points out](https://arxiv.org/pdf/2506.02153).
- Bold, because it breaks with the industry inertia of always relying on a monolithic cloud LLM — the path most apps still follow.

![Screenshot]({{ "/assets/img/ai-architecture.png" | absolute_url }})

By betting on SLMs now, we will not only reduce costs and latency, but also align with what is likely to become the standard paradigm for reliable, sustainable agentic systems.

For implementing this solution, the Google’s [MediaPipe LLM Inference API](https://ai.google.dev/edge/mediapipe/solutions/genai/llm_inference) enables fast, offline calls to a language model directly on Android. In addition, there is a significant [list of small models](https://huggingface.co/litert-community) available, and an active community behind them.

**Are Koog and MediaPipe equivalent / do they overlap?**

- **Not equivalent**: They target different problems. Koog Framework is about building agentic logic / workflows / reasoning; MediaPipe LLM Inference API is the layer that loads a model on the phone, manages memory, and executes text generation: it’s the [basic building block](https://www.anthropic.com/engineering/building-effective-agents) used for the agentic systems like Koog.
- **Complementary rather than redundant**: In many complex AI systems, you can combine: MediaPipe (or parts of it) to process sensor / vision input, then feed results into a Koog agent for decision / reasoning / response.

### Prototype Design:

For an initial prototype I assumed:

- Inference target: local SLM with a possible switch to remote in future versions
- Core triggers handled by Koog; Koog orchestrates MCP calls (weather, seasonal info, local dishes)
- Primary platform: Android (Jetpack Compose), with modules and KMP for core logic in mind for future versions
- Push notifications are the output surface

A sequence diagram for the agent orchestration could be like this:

![Screenshot]({{ "/assets/img/koog-orchestration-sequence.png" | absolute_url }})

Both the input context and the output answer can be described in DTOs

**Input DTO (from Koog to the language model)**

```ruby
userLocale: string (e.g., es-ES)
country: string (e.g., ES)
mealType: enum {BREAKFAST, LUNCH, DINNER, SNACK, WATER}
alreadyLogged: { breakfast: bool, lunch: bool, dinner: bool, snack: bool, waterTodayMl: int }
timeNow: ISO datetime
quietHours: { startLocal: string, endLocal: string, enabled: bool }
weather: { condition: enum, tempC: number, feelsLikeC: number, rainChancePct: number }
season: enum {WINTER, SPRING, SUMMER, AUTUMN}
localDishes: array of { name: string, mealTypes: enum[], tags: string[], region: string }
motivationLevel: enum {LOW, MEDIUM, HIGH}
dietaryTags: array of strings (optional: vegan, halal, celiac, lactose-free)
recentStreakDays: int
```

**For notification output DTO:**

```ruby
title: string
body: string
category: string (e.g., “meal_reminder”, “hydration”)
language: string
confidence: number (0–1)
abVariant: string (for experiments)
```

### Prototype Implementation:

It may sound incredible, but building a simple prototype with a significant part of the requirements described above, is quite straightforward. There is solid documentation from Google and JetBrains with multiple examples. None of them includes local inference but it’s a matter of time that those two worlds converge.

The [current implementation](https://github.com/monday8am/koogagent) contains a simple screen for downloading the model from a static link, changing some parameters and prompting the downloaded model. It demonstrates the notification engine can ‘_think locally_’ before speaking. The output is a text and a notification message.

Prototype snapshots

### Reflections & Trade-offs:

**_The Koog framework is overkill for a simple operation like querying a model:_** Before using agents, it’s better to find the simplest solution possible, and only increasing complexity when needed. But this project is a base for learning and experimentation; future versions will earn the additional structure.

**_The integration of MediaPipe inference and Koog isn’t smooth:_** MediaPipe LLM inference is session‑based, and model initial load is _slow_, typically happening in the background. On the other hand, Koog samples assume always‑alive remote APIs. A practical fix is to bind MediaPipe’s session lifecycle to the app’s main activity (or a scoped service) and expose a readiness state to Koog.

**_The language model, even downloaded in background, is overkill just for pushing better messages:_** Absolutely, the model should serve multiple on‑device tasks to justify its footprint. Even in that case, output and performance testing is needed on different Android devices.

**_A local dev setup for testing increases the speed:_** The Ollama + Mistral combination takes 30 seconds to install and provides a local LLM for testing Koog agents and Kotlin pipelines in a pure JVM project.

**_Privacy and performance is another interesting topic:_** The system runs inference on-device by default and at this stage, doesn’t send personally identifiable information externally. It could cap the prompt size using compact JSON and caches MCP outputs (like weather or season) to reduce latency and battery usage. Timeouts and fallbacks ensure reliability, and an optional remote model is available only with explicit user opt-in.

**_Is Multiplatform a good added value?:_** Not in the first stage of the prototype. Although the Koog related classes are in a separate module, I consider it’s better to invest efforts experimenting with the agentic structure and add the local inference in iOS once the Android side is ready and tested.

### Testing and Evaluation:

As a double wink to product managers, the whole feature can be tested using the following approach:

- A small but representative “golden set” of 50 context scenarios (combining meal type, time, weather, region, and dietary tags) to verify the system responds appropriately across typical edge cases.
- Linguistic checks — length, emoji count, locale, forbidden words, and sensitive claims — to ensure messages are safe, readable, and culturally consistent.
- For impact, A/B test template-based notifications versus LLM-generated ones, measuring tap-through (CTR) and time-to-log to confirm real user benefit.
- Finally, we could enforce an end-to-end latency budget of roughly 250–600 ms on mid-range devices, quantized ~1B SLM, short prompts, and constrained decoding; if the pipeline exceeds that threshold, it will fall back to deterministic templates to preserve UX reliability.

### What is Next?

* Injecting real context (weather MCP, season, local meals)
* Prompt finetuning
* Second model call for translation
* Post-processor and safety checks
* Remote LLM fallback

### Links

- [Prototype app repository](https://github.com/monday8am/lottierecorder)
- [“Building effective agents” from Anthropic](https://www.anthropic.com/engineering/building-effective-agents)
- [JetBrains Koog introduction](https://medium.com/@vadim.briliantov)
- [Google MediaPipe LLM Inference guide](https://ai.google.dev/edge/mediapipe/solutions/genai/llm_inference)
- [Google Edge AI Gallery app repository](https://github.com/google-ai-edge/gallery)
- [HuggingFace SLM models for Android](https://huggingface.co/collections/litert-community/android-models-68c1d00fa08404b27e420a2e)
- [Small Language Models paper from NVIDIA](https://research.nvidia.com/labs/lpr/slm-agents/)
- [Yazio calorie counter](https://www.yazio.com/)
