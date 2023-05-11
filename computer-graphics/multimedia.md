---
title: Multimedia - What It Is, How It Is Made, and Where It Is Used
description: The working definition of multimedia as the computer-controlled integration of text, image, audio, video, and animation, how it evolved from hypertext to VR, the production pipeline, the properties that make a presentation effective, and how analog media becomes digital.
date: 2023-05-11
draft: false
slug: /computer-graphics/multimedia
tags:
  - Computer Graphics
  - Multimedia
---

Multimedia is the point where separate media stop being separate. A textbook shows the solar system in text and pictures; a multimedia lesson adds an orbit animation, a narrator, a model you can rotate, and a quiz that responds to your answers — and a computer coordinates all of it. This post is the overview of that field: the definition, how it got here, how a multimedia piece is built, what makes one effective, and the digitisation step that lets a computer handle any of it. The individual components get their own posts — [text and image](/citadel/computer-graphics/text-image), [audio and video](/citadel/computer-graphics/audio-video), [compression](/citadel/computer-graphics/compression).

## The definition

**Multimedia** is the computer-controlled integration of text, graphics, still and moving images, animation, audio, and any other medium where every type of information can be represented, stored, transmitted, and processed digitally (the phrasing follows Ranjan Parekh's *Principles of Multimedia*). Two words carry the weight: *integration* — the media reinforce one message rather than sitting side by side — and *digital* — everything is numbers, so it can be stored, searched, transmitted, and edited by software.

## How it evolved

- **Foundations.** Vannevar Bush's 1945 "Memex" essay imagined non-linear, associative navigation of information; systems like HyperCard in the 1980s made hypertext real on a personal computer.
- **CD-ROM.** In the 1990s, PCs gained the graphics and sound to run interactive encyclopedias, games, and training titles from disc — Microsoft Encarta being the emblem.
- **The web.** The World Wide Web made multimedia global: pages could embed images, audio, and video, and reach anyone.
- **Mobile and immersive.** Smartphones put multimedia authoring and consumption in a pocket; streaming, social media, augmented reality, and virtual reality push the boundary now.

The trajectory is from static and limited toward dynamic, immersive, and everywhere.

## Presentation versus production

**Presentation** is how the content reaches the user — the finished experience. It might be a website with embedded video, a touch app, a museum kiosk, a computer-based training module, or a live event with projection and sound.

**Production** is the pipeline that builds it:

1. **Planning and design** — goals, audience, content structure, look and feel; storyboards and prototypes.
2. **Asset creation and acquisition** — writing text, designing graphics, recording audio, shooting video, building animation.
3. **Integration and authoring** — combining the assets in authoring software, programming the interactivity, building navigation.
4. **Testing** — bugs, usability, cross-device behaviour.
5. **Delivery and distribution** — deploying the site, publishing the app, shipping the media.

It is collaborative work, drawing on writers, designers, programmers, and subject-matter experts.

## What makes a presentation work

- **Integration** — the media are deliberately combined to reinforce one message.
- **Interactivity** — the user controls their path, makes choices, and gets feedback; this active role is what raises engagement over passive media.
- **Digital representation** — every component is data, so it can be stored, transmitted, and manipulated.
- **Non-linearity** — hyperlinking lets users move through information in the order they choose.
- **Variety** — engaging more than one sense suits more learning styles and makes content more memorable.
- **Immersiveness** — a well-built experience draws the user in.

## The components

| Component | Role |
| --- | --- |
| **Text** | Detailed information, titles, captions, navigation; can be plain, formatted, or hyperlinked |
| **Graphics and images** | Photographs, illustrations, icons, diagrams — as pixel-based **raster** or equation-based **vector** (detailed in [text and image](/citadel/computer-graphics/text-image)) |
| **Audio** | Narration, music, sound effects, ambience |
| **Video** | Moving images with synchronised audio for dynamic storytelling and demonstration |
| **Animation** | The illusion of movement in graphics — explainer sequences, effects, character motion |

Layered on top of all five is **interactivity**, which turns the user from an observer into a participant.

## Hardware and software

**For production:** capable machines — fast CPU and GPU, ample RAM, SSD storage — plus input devices (scanners, cameras, microphones, drawing tablets) and quality output (high-resolution monitors, monitoring speakers). **For consumption:** anything from a gaming PC to a phone, as long as it can decode and display the media used.

Software falls into layers: the operating system; media editors (image — Photoshop, GIMP; audio — Audacity, Audition; video — Premiere Pro, DaVinci Resolve; animation — Adobe Animate, Blender); authoring systems that assemble the assets and add interactivity (Adobe Captivate, Articulate, Unity, web frameworks); and playback software or browsers on the delivery side.

## Digitisation

Real-world signals are continuous; a computer needs discrete numbers. **Digitisation** is that conversion, and it has two steps.

- **Sampling** — measure the analog signal at regular intervals. A higher **sampling rate** (samples per second for audio and video, pixels per unit length for images) captures more of the original, especially its fast-changing parts.
- **Quantisation** — assign each sample a value from a fixed set. The number of bits per sample, the **bit depth**, sets how many values are available and therefore the fidelity: more bits mean more colours in an image or more dynamic range in audio.

These choices decide the quality, file size, and processing cost of every multimedia element, and they are the starting point for [compression](/citadel/computer-graphics/compression). The image-specific and audio-specific forms are covered in [image processing](/citadel/computer-graphics/image-processing) and [audio and video](/citadel/computer-graphics/audio-video).

## Where it is used

- **Education and training** — e-learning modules, educational games, virtual labs, simulations.
- **Entertainment** — games, streaming film and television, interactive fiction, VR and AR.
- **Business and e-commerce** — interactive ads, product demos, 360° views, virtual showrooms, video conferencing.
- **Information and reference** — news portals, digital encyclopedias, museum kiosks, navigation systems.
- **Communication** — social media, video calls, interactive messaging.
- **Creative arts** — digital art, music production, filmmaking, installations.
- **Science and medicine** — medical imaging, surgical simulation, data visualisation, virtual anatomy.

## The one idea to keep

Multimedia is not a list of media types; it is their coordinated integration under software control, made possible by one step — digitisation — that turns pictures, sound, and motion into numbers a computer can store and combine. What separates an effective piece from a cluttered one is that the media are chosen to reinforce a single message and the user has something to do.
