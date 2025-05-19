---
layout: post
title:  "Storybeat tales"
date:   2021-11-21 21:12:00 +0100
categories: blog
---

### Storybeat tales

[Storybeat](https://www.storybeat.com/) is a popular video editor packed with templates, filters, and creative tools. When I joined the team (just two developers), the app was a basic MVP built on top of [FFmpeg](https://github.com/tanersener/mobile-ffmpeg): too slow when rendering videos and with no real path for future evolution. The challenge was to rewrite it from scratch, improving rendering times and adding a real video editor.

I built an OpenGL-based video editor connected to a custom video recording pipeline entirely from scratch. Since the editor runs with OpenGL on a full-screen TextureView, all typical WYSIWYG operations (move, flip, zoom, color adjustments) are performed directly on 2D OpenGL textures using transformation matrices and shaders.

Why introduce this level of complexity instead of using the standard Android View framework? The answer is simple: **performance and memory efficiency**.

Once the OpenGL context is initialized and resources are loaded, rendering can happen either on the TextureView (for real-time editing) or on a SurfaceTexture provided by the recording pipeline. This allows seamless switching between editing and recording modes without increasing memory usage. Best of all, the output matches exactly what the user sees on screen.

Sometimes, the right choice is to invest in a complex, low-level foundation for a critical part of the app, so the rest of the codebase stays clean, stable, and feature-rich. Three years later, Storybeat has grown to over **10 million downloads** and holds a **4.5-star rating from more than 300,000 reviews**. I’m incredibly proud of the work I did there.


#### Links

- [Storybeat](https://www.storybeat.com/)
- [OpenGl Canvas Lib](https://github.com/ChillingVan/android-openGL-canvas)