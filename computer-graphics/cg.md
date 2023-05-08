---
title: Computer Graphics - Pixels, Displays, and Input Devices
description: What a digital picture is made of, how the major display technologies turn pixel values into light, the difference between raster-scan and vector displays, and the input primitives and devices that let you draw into a graphics system.
date: 2023-05-08
draft: false
slug: /computer-graphics/cg
tags:
  - Computer Graphics
  - Rendering
  - Multimedia
---

A digital image is a grid of numbers, and a screen is a machine for turning that grid into light. Everything in computer graphics sits between those two facts: how the numbers are chosen (drawing and rendering), how they are transformed (geometry), and how the hardware displays and captures them. This post is the ground floor — what a picture is made of, how displays work, and how you get your intent into the system in the first place.

**Computer graphics** is the branch of computing concerned with creating, manipulating, and displaying visual information. It runs from the icon on your phone to a feature-length animated film, and it shows up in:

- **Entertainment** — films, games, animated series.
- **Design and engineering** — architects and engineers work in **computer-aided design** (CAD); **computer-aided manufacturing** (CAM) drives machines from those models.
- **User interfaces** — every button and menu is drawn.
- **Medical imaging** — MRI, CT, and ultrasound reconstruction and display; surgical simulators.
- **Scientific and data visualisation** — turning tables of numbers into charts and 3D models.
- **Education and training** — virtual labs, flight simulators, VR walkthroughs.

## What a picture is made of

**Pixels.** Most images on screens are **raster graphics** — a grid of small coloured squares called pixels ("picture elements"), each with a colour and brightness. The image is a mosaic; more tiles mean more detail.

**Resolution.** How many pixels an image or display has, written width × height — 1920 × 1080 is "Full HD". More pixels in the same area gives a sharper image. Approximating a circle with a few large squares looks blocky; with thousands of small ones it looks smooth.

**Geometric primitives.** The basic shapes a graphics system can draw:

- **Point** — a location, $(x, y)$ in 2D or $(x, y, z)$ in 3D.
- **Line segment** — a straight path between two points.
- **Polygon** — a closed shape of connected segments. Triangles matter most in 3D: they form the surface skin of complex objects.
- Curves (Bézier and others), circles, ellipses, and 3D solids (spheres, cubes, cylinders) build on these.

## Display technologies

Every display turns a pixel value into emitted or transmitted light; they differ in how.

| Technology | How it makes light | Trait |
| --- | --- | --- |
| **CRT** | Electron gun fires a beam at a phosphor coating that glows where hit | Bulky, power-hungry; historical |
| **LCD** | A backlight shines through liquid crystals that twist under voltage, acting as per-subpixel shutters over red/green/blue filters | Slim, efficient |
| **LED** | Usually an LCD with LEDs as the backlight; "direct-view LED" (stadium screens, emerging MicroLED) uses one LED per pixel | Brighter, thinner than fluorescent-backlit LCD |
| **OLED** | Each pixel is an organic material that emits its own light | Perfect blacks (pixel off), high contrast, flexible |
| **QLED** | An LCD with a quantum-dot layer that emits pure colours when lit by the backlight | Wider colour gamut and brightness than plain LCD |

## Raster scan versus vector displays

How does the computer instruct the screen? Two historical answers:

**Raster scan** — the method in every modern display. The image lives as a grid of pixel values in the **frame buffer**, a region of memory holding a complete snapshot of the screen. The display hardware reads the frame buffer and paints the screen one horizontal row of pixels — one **scan line** — at a time, top to bottom, and repeats the whole screen many times per second. That rate is the **refresh rate** (60 Hz is common), and the eye perceives the rapid redraw as a steady image.

**Random scan (vector) displays** — used in early CAD workstations and vector arcade games like *Asteroids*. Instead of filling every pixel, the electron beam is steered directly from point to point, drawing lines. The image is stored as a **display list** of line-drawing instructions. Vector displays produce very sharp, alias-free lines and are efficient for line art, but they cannot show filled or shaded areas well. The idea survives in vector file formats like SVG, which describe an image as shapes and equations rather than a pixel grid.

## Input primitives and devices

Graphics is interactive, so a system needs ways to receive intent. The **input primitives** are the kinds of data you provide:

- **Locator / position** — a coordinate, for drawing or placing.
- **Stroke** — a sequence of positions, like a freehand curve.
- **String** — text.
- **Valuator** — a single number, such as an angle or a slider value.
- **Choice** — selecting an option from a menu.
- **Pick** — selecting a displayed object.

The **input devices** that supply them:

| Device | Primitives it provides |
| --- | --- |
| Keyboard | String; choice (shortcuts) |
| Mouse | Locator (cursor); choice and pick (clicks) |
| Trackball / trackpad | Locator |
| Joystick / gamepad | Valuator (direction, speed); choice (buttons) |
| Touch screen | Locator, stroke, pick; gestures |
| Stylus + graphics tablet | High-precision locator and stroke, often pressure as a valuator |
| Scanner | Converts a physical document to a raster image |
| Digital camera / webcam | Captures real-world images |
| Microphone | Voice, mapped to commands or text |
| 3D scanner | Captures object geometry as a 3D model |

## The one idea to keep

A picture is a grid of numbers and a screen is a device that lights up that grid, scan line by scan line, out of the frame buffer. Displays differ only in how a pixel value becomes light — a shuttered backlight, a self-emitting pixel, a steered electron beam. Everything else in computer graphics is about choosing and transforming the numbers before they reach that buffer.
