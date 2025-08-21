---
layout: post
title:  "Rendering a video from a Lottie animation "
date:   2021-11-21 21:12:00 +0100
categories: blog
---


# 🎥 Exporting a Lottie Animation to Video with Audio Using Media3

Exporting a Lottie animation to an MP4 file — *with audio* — is a powerful feature for any creative or media-rich Android app. Whether you’re building a story editor, motion graphics tool, or onboarding animation, the ability to render Lottie animations as sharable videos can significantly enhance your user experience. Besides, it's a common case inside very succesful applications like [Unfold](https://unfold.com/), [InStories](https://instories.com/) or [Storybeat](https://www.storybeat.com/) that provides thousands of templates with animations to the users and the tools for costumizing them adding photos, text or videos.

This post walks you through a clean and performant pipeline that renders Lottie to video using the [Media3 Transformer API](https://developer.android.com/media/transformer/overview) and OpenGL — and adds audio support along the way.

---

## What We'll Build

We'll implement a function:

```kotlin
suspend fun recordLottieToVideo(
  context: Context,
  lottieFrameFactory: LottieFrameFactory,
  audioUri: String,
  outputFilePath: String,
  onProgress: (Float) -> Unit = {},
  onSuccess: (fileSize: Long) -> Unit = {},
  onError: (Throwable) -> Unit = {},
  durationMs: Long,
  frameRate: Int = 30,
  width: Int = 1080,
  height: Int = 1920
)
```

This function:

* Renders Lottie frames to GPU textures
* Encodes them to video using Media3
* Adds an audio track via raw PCM decoding
* Outputs a complete `.mp4` file

---

## The Problem: Bridging Two Worlds

After some research, the solution to exporting Lottie animations with audio using Media3 comes down to connecting two seemingly unrelated pieces:

* On one side, we have LottieDrawable, a powerful engine that can draw animations onto any Canvas, including hardware-accelerated ones.

* On the other side, we have Media3's Transformer, which can be configured with a RawAssetLoader to receive custom video frames (as OpenGL texture IDs) and raw audio chunks.

But how do you go from a LottieDrawable — something that draws on a Canvas — to an OpenGL texture ID that Media3 can encode?

The missing link is ImageReader.

ImageReader gives us a Surface that:

Can be drawn to using lockHardwareCanvas() (GPU-accelerated)

Provides Image frames that can be uploaded to the GPU as textures

This makes it the perfect bridge between the CPU-based Lottie drawing world and the GPU-based Media3 encoding pipeline.

LottieDrawable --draws to--> ImageReader.surface (via lockHardwareCanvas)
ImageReader --outputs--> Image (GPU-compatible)
Image --uploaded to--> GL Texture
Texture --queued to--> RawAssetLoader --encoded by--> Media3 Transformer





### Step 1: Control Everything with `RawAssetLoader`

The `RawAssetLoader` gives you full manual control over how video and audio data is fed into the Media3 encoder:

```kotlin
.setAssetLoaderFactory(
  RawAssetLoaderFactory(
    audioFormat = audioFormat,
    videoFormat = videoFormat,
    onRawAssetLoaderCreated = { loader -> rawAssetLoaderDeferred.complete(loader) }
  )
)
```

From here on, you're responsible for queuing:

* OpenGL texture IDs with timestamps
* Raw PCM audio chunks with timestamps

The audio chunks can be read from a local audio file in the traditional, using a `MediaExtractor`, a `MediaCodec` and creating a pipeline of small bufferes that are extracted, decoded and then passd to the `RawAssetLoader`'s callback.


---

### Step 2: Render Lottie to a Hardware Canvas

Each frame of the Lottie animation is generated as a `Drawable` using a LottieDrawable. The most performant way to get that drawable into a video frame is to draw it into a hardware-accelerated canvas — not into a regular software Bitmap.

```kotlin
val hCanvas = imageReader.surface.lockHardwareCanvas()
try {
  lottieFrame.setBounds(0, 0, videoWidth, videoHeight)
  lottieFrame.draw(hCanvas)
} finally {
  imageReader.surface.unlockCanvasAndPost(hCanvas)
}
```

This approach is not just efficient — it’s the key to real-time performance. By using lockHardwareCanvas() from an ImageReader.Surface, you automatically get a hardware-backed Canvas that is connected to GPU memory.

### ✅ Does Lottie use the GPU here?

Yes! Internally, `LottieDrawable.draw(Canvas)` chooses between a software (`renderAndDrawAsBitmap`) and hardware (`drawDirectlyToCanvas`) rendering path.

If the `Canvas` is hardware-accelerated — as it is when created from `lockHardwareCanvas()` — Lottie will automatically use GPU rendering via `drawDirectlyToCanvas()`.

---

## 🔄 Step 3: Upload Frame as Texture via `ImageReader`

After unlocking the canvas, we acquire the most recent frame from the `ImageReader` and upload it to OpenGL as a texture:

```kotlin
val image = awaitForLastImage.await()
val textureId = uploadImageToGLTexture(image)

rawAssetLoader.queueInputTexture(textureId, presentationTimeUs)
```

This texture now represents a video frame and will be encoded by Media3.

---

## 🔊 Step 4: Decode and Enqueue Audio

In parallel, we decode audio from a URI into raw PCM chunks using a custom `AudioDecoder`. Each chunk is queued manually into the asset loader:

```kotlin
val (audioChunk, presentationTimeUs) = awaitForAudioChunk!!.await()

rawAssetLoader.queueAudioData(audioChunk, presentationTimeUs, isLast)
```

Timing coordination is up to you — the power (and responsibility) is yours.

---

## 🧵 Full Pipeline Example

```kotlin
val transformer = Transformer.Builder(context)
  .setAssetLoaderFactory(...)
  .build()

val composition = ... // load your LottieComposition

transformer.start(editedMediaItem, outputFilePath)

for (frameIndex in 0 until totalFrames) {
  val drawable = lottieFrameFactory.generateFrame(frameIndex)
  // Draw to Surface → Acquire Image → Convert to Texture
  rawAssetLoader.queueInputTexture(textureId, timestamp)
}

// Decode and queue audio PCM chunks
```

---

## ✅ Why This Approach Works

* `RawAssetLoader` gives full audio+video control
* `ImageReader` is optimized for canvas-to-texture use
* `lockHardwareCanvas()` ensures GPU memory drawing
* `LottieDrawable` already handles GPU rendering internally when used on a hardware canvas

---

## 🚀 Future Improvements

* Add progress reporting (`onProgress: (Float) -> Unit`)
* Use a `SurfaceTexture` + GL rendering instead of `Canvas` for full GPU control
* Investigate using [Skottie (Skia GPU Lottie renderer)](https://skia.org/docs/user/modules/skottie/) for even faster native rendering

---

## 📂 References

* [Media3 Transformer API](https://developer.android.com/media/transformer/overview)
* [Airbnb Lottie Android](https://airbnb.io/lottie/#/)
* [ImageReader API](https://developer.android.com/reference/android/media/ImageReader)
* [`lockHardwareCanvas()`](https://developer.android.com/reference/android/view/Surface#lockHardwareCanvas%28%29)

---

Let me know if you'd like to see the full codebase or a working example project!


In this article I'll introduce how to render a mp4 video from a Lottie animation and add a custom audio. It's a common case inside very succesful applications like Unfold, InStories or Storybeat that provides thousands of templates with animations to the users and the tools for costumizing them adding photos, text or videos. 

Let's guess in one side we have the Lottie animation we want to export as a video and we have created a wrapper that permits to extract a lottie frame frame passing the frame number. This wrapper could implement the following interface:

[LottieFrameFactory interface code]

Calling factory.getFrame(xxx) will returns a LottieDrawable object. It's a native Drawable and we can call the native draw method passing a Canvas to capture the content as a bitmap. 
And here is a crucial requirement, this operation must be fast to get a low recording time, and the best way to achieve it is using the hardware-accelerated lockHardwareCanvas option. 

On the other side, we have the new Media3 Transform API that permits mix and recording a video from multiple types of sources. 


Which existing API permits



In one side, we have a bitmap factory created from a Lottie file (LottieFrameFactory) that returns a bitmap passing a frame number. 

On the other side, we find a RawAssetLoader that accepts OpenGL texture ids. We're forced to use a RawAssetLoader because it's the only Transformer configuration setup that permits mixing a custom video and audio sources at the same time. A SurfaceAssetLoader would simplify everything but then, at least from my tests, an audio can't be added.

How to magically "transform" a raw bitmap into an OpenGL texture id? The first step is creating an ImageReader and draw the bitmap into the ImageReader's surface. That's a critical point, due to performance requirements, the bitmap should be drawn using the lockHardwareCanvas, and ImageReader returns the type of Surface permitted for that operation. Once the bitmap is drawn, it's uploaded to an OpenGL texture and passed to the Transformer's asset loader.

I have used the coroutine CompletableDeferred calls as semaphores to suspend the thread and wait for the corresponding results in a typesafe way: the assetLoaderDeferred, awaitReadyForInput and awaitForLastImage controls the flow and makes the whole operation a suspend function. This design also makes it easy to wrap the process inside a use case that emits a Flow of results.

I'm sorry for presenting my example code in this informal way, I hope you enjoy it. I enjoyed as well doing it, mostly realizing how the RawAssetLoader works because there's no documentation at all :) . Lottie is a wonderful animation framework, and I’m considering writing a follow-up on creating the LottieFrameFactory with dynamic bitmaps. 

Links:



#### Links

- [LottieDrawable](TBD)
- [Media3 Transformer](https://developer.android.com/reference/kotlin/androidx/media3/transformer/Transformer)
- [SurfaceAssetLoader](https://developer.android.com/reference/androidx/media3/transformer/SurfaceAssetLoader)
- [ImageReader](https://developer.android.com/reference/android/media/ImageReader)
- [Image](https://developer.android.com/reference/android/media/Image)
- [CompletableDeferred](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-completable-deferred/)

