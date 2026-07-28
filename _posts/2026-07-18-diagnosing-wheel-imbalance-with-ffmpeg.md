---
layout: post
title: "Diagnosing Wheel Imbalance with a GoPro and FFmpeg"
date: 2026-07-18 10:00:00 -0400
categories: [cars, hardware-hacking]
tags: [ffmpeg, gopro, maintenance]
excerpt: "Using frame differencing in FFmpeg to visually isolate vehicle vibrations."
---

There is nothing quite as annoying as a highway-speed vibration that you just can't track down. You've had the wheels balanced, the alignment checked, and yet, at 70 MPH, the steering wheel still dances. 

Recently, I decided to stop guessing and start measuring. Using a GoPro mounted under the car and some `ffmpeg` magic, I was able to visually isolate exactly which wheel was causing the problem.

Here is the nerd write-up on how to use frame differencing to diagnose mechanical vibrations.

## The Setup

I mounted a GoPro to the frame of the car, pointed at the suspension assembly. The goal was to capture high-frame-rate footage (60fps or higher is best) of the wheel while driving at the speed where the vibration is most prominent.

![GoPro Under Car Setup](/assets/images/posts/wheel-imbalance/img_2416.jpg)

Once I had the footage, the real work began.

## Phase 1: Metadata and Overview

First, we need to understand what we're working with. `ffprobe` is the go-to for extracting metadata.

```bash
ffprobe -v error -show_format -show_streams input_video.mov
```

I also like to generate a contact sheet of thumbnails to quickly find the sections of video where I was actually at highway speed.

```bash
mkdir -p thumbnails
ffmpeg -i input_video.mov -vf "fps=1/60" thumbnails/thumb_%03d.jpg
```

## Phase 2: Visual Evidence

Before diving into advanced filters, a simple scaled-down GIF can help confirm you have the right angle.

```bash
ffmpeg -ss 00:05:30 -t 5 -i input_video.mov -vf "scale=480:-1" -r 15 -f gif observation.gif
```

![Driver Side Observation](/assets/images/posts/wheel-imbalance/video1_later.gif)

## Phase 3: Advanced Diagnosis (Vibration Isolation)

This is where it gets interesting. To prove that a movement is a **mechanical vibration** rather than just a **road bump**, we use **Frame Differencing**.

Road bumps are irregular and usually affect the whole frame. A mechanical imbalance is periodic and constant. By subtracting the previous frame from the current one, we can highlight only the pixels that are changing.

### The Magic Command

```bash
ffmpeg -y -ss [start_time] -t 3 -i input_video.mov \
  -vf "format=gray, tblend=all_mode=difference, scale=480:-1" \
  -r 15 -f gif vibration_diff.gif
```

- `format=gray`{:.language-bash}: Strips color to focus on intensity.
- `tblend=all_mode=difference`{:.language-bash}: The core logic. It highlights changes between consecutive frames.

### Interpreting the Results

In a difference map:
- **Road Bumps:** Appear as sudden, temporary flashes of the whole assembly.
- **Mechanical Vibration:** Appears as a constant "ghosting" or "fuzziness" around the edges of the wheel and suspension.

#### Driver Side (The Control)

For comparison, here is the driver side. The edges are much cleaner, and the "ghosting" is significantly reduced.

![Driver Side Vibration Diff](/assets/images/posts/wheel-imbalance/driver_vibration_diff.gif)
*The difference map is mostly black until the end when you see the effect of hitting a bump.*

#### Passenger Side (The Problem)

Before applying any noise reduction, a simple difference map shows the passenger side, but it is extremely noisy due to compression artifacts and camera sensor noise:

![Passenger Side Simple Vibration Diff](/assets/images/posts/wheel-imbalance/passenger_vibration_diff_simple.gif)
*A simple frame difference without thresholding is flooded with low-level noise.*

To clean this up and isolate the true mechanical vibration, we apply a threshold to the filter chain. By adding `lutyuv=y='if(gt(val,20),val,0)'`{:.language-bash} (or `if(gt(val,20),255,0)`{:.language-bash} for binary contrast), we drop any pixel intensity change below a value of 20 to zero (black).

Here is the much cleaner result highlighting only the persistent high-frequency oscillation around the tire and control arms:

![Passenger Side Vibration Diff](/assets/images/posts/wheel-imbalance/passenger_vibration_diff.gif)
*The thresholded difference map cleanly highlights the mechanical oscillation.*

## The Final Result: Visual Telemetry

By overlaying the difference map back onto the original footage and colorizing it (magenta, in this case), we get what I call "visual telemetry." 

To make this effective, I used a **threshold filter** to strip away low-level road noise and only highlight the most intense mechanical vibrations. 

### The Visual Telemetry Command

This is the multi-step filter chain required to isolate the movement and paint it magenta:

```bash
ffmpeg -y -i observation.gif -i vibration_diff.gif -filter_complex \
  "[1:v]format=gray,lutyuv=y='if(gt(val,20),255,0)',format=rgb24,lutrgb=g=0[magenta_mask]; \
   [0:v][magenta_mask]blend=all_mode=lighten,split[s0][s1]; \
   [s0]palettegen[p]; [s1][p]paletteuse" vibration_overlay.gif
```

**How it works:**
1.  **Thresholding:** `lutyuv=y='if(gt(val,20),255,0)'`{:.language-bash} takes the grayscale difference map and turns any pixel with a value over 20 into pure white, and anything else into pure black. This removes the "noise" of small road bumps.
2.  **Colorizing:** `lutrgb=g=0`{:.language-bash} strips the green channel from the white mask, leaving us with solid magenta.
3.  **Blending:** `blend=all_mode=lighten`{:.language-bash} overlays our magenta highlights onto the original footage, only keeping the magenta pixels if they are brighter than the background (which they are).

![Vibration Overlay](/assets/images/posts/wheel-imbalance/vibration_overlay.gif)

Notice how the magenta "glow" is concentrated on the tire tread and the suspension components. This is the smoking gun for a mechanical imbalance. Since we've thresholded the output, only the high-frequency oscillation from the imbalance is captured in the overlay.

## Conclusion

After analyzing the visual telemetry, I pulled off the passenger wheel for a physical inspection. The culprit was immediately clear: the passenger wheel had lost a wheel weight, becoming imbalanced at highway speeds. Once the wheel was rebalanced at the shop, the vibration was completely gone.

Using `ffmpeg` for this kind of visual telemetry is incredibly powerful. Instead of throwing parts at a problem, you can see the physical manifestation of the issue and isolate the root cause before taking things apart. Next time your car has a mysterious shake, grab a GoPro and start differencing.

