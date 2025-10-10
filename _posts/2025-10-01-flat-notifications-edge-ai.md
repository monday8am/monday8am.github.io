---
layout: post
title:  "From flat Notifications to Edge AI"
date:   2025-10-01 14:23:00 +0100
categories: blog
---

## Introduction

For the last few weeks I have been testing the [**Yazio**](https://www.yazio.com/) app, a calorie counter. While using it, I noticed that the notifications — though helpful — were sometimes similar and easy to ignore. This observation sparked a question:

_Could these reminders be generated dynamically, on the device, based on my context, and sound more natural or timely?_

That question led me into an experiment that sits at the intersection of two emerging technologies: **agentic frameworks** (JetBrains Koog) and **on-device small language models** (Google MediaPipe + quantized LLMs). While both exist independently, their integration is still uncharted territory.

The result? A [foundational prototype](https://github.com/monday8am/koogagent) that proves these pieces can work together on Android. This post documents the architecture decisions, integration challenges, what I learned building it and what's are my next steps.

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

```text
Time for a snack?
```

vs.

```text
Nice pace today. Since lunch is logged, a quick 
summer bite — gazpacho or yogurt with peach — will 
keep you moving 💪
```

It could feel more personal and relevant — not by guessing, but by _responding to context_. Not quite like your mom, but better than a static reminder.

Hardcoding these messages isn’t scalable, but solving problems of generated language and _nondeterministic outputs_ is exactly where language models shine. So how do we build this context into an app? That's where agent frameworks come in.

## The Solution: Agents + SLM

Building context-aware notifications requires solving two distinct problems:

1. **Reasoning and orchestration:** Deciding *when* to notify, *what* context to gather, and *how* to structure the information
2. **Natural language generation:** Converting that structured context into a human-friendly message

The natural tool choices are [agentic frameworks](https://www.ibm.com/think/insights/top-ai-agent-frameworks) for the reasoning layer and **language models** for the generation layer:

### Using Koog to Emulate an Agent

In case of the Android platform, [JetBrains Koog](https://github.com/JetBrains/koog) is the valid choice. It’s created in Kotlin and instead of making a raw prompt request, Koog helps implement the following loop:

![Screenshot]({{ "/assets/img/koog-agentic-flow.png" | absolute_url }})

Koog provides connectors for accessing to data using [Model Context Protocol](https://www.anthropic.com/news/model-context-protocol) (MCP) and common large language model APIs, orchestrating tools and decisions around a model. 

### The Edge AI choice

For integrating a model, I believe an _hybrid architecture_ is the best solution: small language models ([SLMs](https://huggingface.co/blog/jjokah/small-language-model)) for the default local workflow, with a large model in the cloud as a fallback. 

![Screenshot]({{ "/assets/img/ai-architecture.png" | absolute_url }})

By betting on SLMs now, we will not only reduce costs and latency, but also align with what is likely to become the standard paradigm for reliable, sustainable agentic systems.

For implementing this solution, the Google’s [MediaPipe LLM Inference API](https://ai.google.dev/edge/mediapipe/solutions/genai/llm_inference) enables fast, offline calls to a language model directly on Android. In addition, there is a significant [list of small models](https://huggingface.co/litert-community) available, and an active community behind them.

### The Integration Gap

But here's the catch: **these two worlds haven't been integrated yet.**

- Agentic frameworks assume always-available cloud APIs with instant responses
- On-device inference APIs (like MediaPipe) assume simple request/response patterns with session management
- Koog's examples use OpenAI/Anthropic/Ollama — none target mobile SLMs
- MediaPipe's examples focus on standalone inference, not multi-step reasoning

The gap isn't theoretical—it's architectural. Session lifecycles, latency profiles, error handling patterns, and resource constraints are fundamentally different between cloud LLMs and on-device SLMs.

**This experiment explores whether that gap is bridgeable**, and if so, what the integration patterns look like.

## Prototype design and implementation

### Scope

Currently Implemented:
- Model download and initialization (bundled in repository for simplicity)
- Koog agent structure set up with basic orchestration
- Local inference execution with a hardcoded context example
- Proof that an SLM can generate notification-style text on-device
- Push notifications as the output surface

Not yet implemented (but architected):
- MCP integration for real-time weather/season data
- Safety checks and content filtering
- Remote LLM fallback
- Multi-platform support (iOS)
- User activity tracking and logging

### Design

**Sequence diagram for the agent orchestration:**

![Sequence flow: meal reminder agent orchestration]({{ "/assets/img/koog-orchestration-sequence.png" | absolute_url }})

<details>
  <summary>Full diagram source (Mermaid)</summary>
    {% gist 71109a5888d205df4508498487fc286e %} 
</details>
<br />

**Model Choice:**  
The prototype uses the [gemma-3n-E2B-it-litert-lm](https://huggingface.co/google/gemma-3n-E2B-it-litert-lm) model from Google. This size balances capability with mobile constraints—small enough to download over WiFi, large enough for coherent text generation.

**Model Distribution:**  
Rather than requiring users to authenticate with HuggingFace or configure API keys, the model is bundled in the GitHub repository as a [zip file](https://github.com/monday8am/koogagent/releases/download/0.0.1/gemma3-1b-it-int4.zip). This approach prioritizes developer experience for a learning prototype:
- Clone → Build → Run immediately
- No external dependencies or account creation
- Full reproducibility without network calls
- WorkManager for background download

**Input DTO example (Koog -> SLM):**
```ruby
userLocale: string (e.g., es-ES)
country: string (e.g., ES)
mealType: enum {BREAKFAST, LUNCH, DINNER, SNACK, WATER}
alreadyLogged: { breakfast: bool, lunch: bool... }
weather: { condition: enum, tempC: number, feelsLikeC: number }
season: enum {WINTER, SPRING, SUMMER, AUTUMN}
localDishes: array of { name: string, mealTypes: enum[], }
motivationLevel: enum {LOW, MEDIUM, HIGH}
```

**Output DTO example (SLM -> Notification):**
```ruby
title: string
body: string
category: string (e.g., “meal_reminder”, “hydration”)
language: string
confidence: number (0–1)
```
**System prompt example:**
```text
You are an nutritionist that generates short, motivating 
reminders for logging meals or water intake.
```

**User prompt example:**
```text
Context:
- Meal type: {mealType}
- Motivation level: {motivationLevel}
- Weather: {weather}
- Already logged: {alreadyLogged}
- Language: {userLocale}
- Country: {country}
Generate a title (max 35 characters) and a body 
(max 160 characters) in plain JSON  format: 
{"title":"...", "body":"...", "language":"en-US", 
"confidence":0.9} Use the language and suggest a meal or drink
based on the country provided. {if (alreadyLogged) "- The user
has already logged something today." else "the user has not 
logged anything today."}
``` 

### Implementation

It may sound incredible, but building a simple prototype with a significant part of the requirements described above, is quite straightforward. There is solid documentation from Google and JetBrains with multiple examples. None of them includes local inference but it’s a matter of time that those two worlds converge.

The [current implementation](https://github.com/monday8am/koogagent) contains a simple screen for downloading the model from a static link, changing some parameters and prompting the downloaded model. It demonstrates the notification engine can ‘_think locally_’ before speaking. The output is a text and a notification message.

![Prototype screenshots]({{ "/assets/img/koog-prototype.png" | absolute_url }})

**The MediaPipe/Koog Integration**

MediaPipe LLM inference is session-based with slow initial loads, typically happening in the background. On the other hand, Koog's examples assume always-alive remote APIs with sub-second response times. 

I have used the following flow to solve this issue:

- Handle async model loading before Koog can invoke it and get a LLM session reference. A practical fix is _binding MediaPipe's session lifecycle to the app's main activity_ (or a scoped service).
- Integrates classes from both frameworks using the LLM session
<details>
  <summary>Integration code example (click to expand)</summary>
  {% gist d0e11f7514aee15e487cca2f4bc25129 %}
</details>

- Manages timeouts and failures gracefully

## Reflections & Trade-offs

**_The Koog framework is overkill... for now:_** Before using agents, it's better to find the simplest solution possible, only increasing complexity when needed. Right now Koog's full orchestration layer is _unnecessary overhead_. However, this project is a foundation for _learning and experimentation_. Future versions that incorporate tool calls will justify the additional structure.

**_The language model, even downloaded in background, is overkill just for pushing better messages:_** Absolutely. Even downloaded in the background, a 500MB model that consumes 600MB RAM is absurd if only used for push messages. This same model should serve multiple on-device tasks:
- Contextual notifications (this prototype)
- Voice input processing and summarization
- Offline chat or FAQ responses
- Personalized content suggestions

**_A local dev setup for testing increases the speed:_** The Ollama + Mistral combination takes 30 seconds to install and provides a local LLM for testing Koog agents and Kotlin pipelines in a pure JVM project.

**_Privacy and performance is another interesting topic:_** The system runs inference on-device by default and at this stage, doesn’t send personally identifiable information externally. It could cap the prompt size using compact JSON and caches MCP outputs (like weather or season) to reduce latency and battery usage. Timeouts and fallbacks ensure reliability, and an optional remote model is available only with explicit user opt-in.

**_Is Multiplatform a good added value?:_** Not in the first stage of the prototype. Although the Koog related classes are in a separate module, I consider it’s better to invest efforts experimenting with the agentic structure and add the local inference in iOS once the Android side is ready and tested.

## Testing and Evaluation:

Thinking in product managers, the whole feature can be tested using the following approach:

- A small but representative “golden set” of 50 context scenarios (combining meal type, time, weather, region, and dietary tags) to verify the system responds appropriately across typical edge cases.

- Linguistic checks — length, emoji count, locale, forbidden words, and sensitive claims — to ensure messages are safe, readable, and culturally consistent.

- For impact, A/B test template-based notifications versus LLM-generated ones, measuring tap-through (CTR) and time-to-log to confirm real user benefit.

- Finally, we could enforce an end-to-end latency budget of roughly 250–600 ms on mid-range devices, quantized ~1B SLM, short prompts, and constrained decoding; if the pipeline exceeds that threshold, it will fall back to deterministic templates to preserve UX reliability.

**Current state:** These metrics are hypothetical. The prototype doesn't yet collect telemetry or implement A/B testing infrastructure. This section documents how I *would* evaluate the system in production.

## Conclusions

This integration represents a path toward **sustainable, privacy-preserving agentic AI**. Instead of sending every user interaction to cloud LLMs, we can reason and generate locally for routine tasks—reserving expensive cloud calls for truly complex problems. 

The prototype is incomplete and over-engineered for its current capabilities. But it proves the foundation exists.

### What is Next?

1. **Real context integration** — Connect MCP servers for weather, season, and local dish data instead of hardcoded examples
2. **Prompt refinement** — Iterate on prompt structure based on output quality
3. **Safety layer** — Implement post-processing checks before notification delivery
4. **Remote fallback** — Add cloud LLM fallback with explicit user opt-in for complex scenarios
5. **Performance profiling** — Comprehensive benchmarking across device tiers and Android versions

### Links

- [Prototype app repository](https://github.com/monday8am/lottierecorder)
- [“Building effective agents” from Anthropic](https://www.anthropic.com/engineering/building-effective-agents)
- [JetBrains Koog introduction](https://medium.com/@vadim.briliantov)
- [Google MediaPipe LLM Inference guide](https://ai.google.dev/edge/mediapipe/solutions/genai/llm_inference)
- [Google Edge AI Gallery app repository](https://github.com/google-ai-edge/gallery)
- [HuggingFace SLM models for Android](https://huggingface.co/collections/litert-community/android-models-68c1d00fa08404b27e420a2e)
- [Small Language Models paper from NVIDIA](https://research.nvidia.com/labs/lpr/slm-agents/)
- [Yazio calorie counter](https://www.yazio.com/)
