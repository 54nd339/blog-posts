---
title: Multimedia Architecture - APIs, Streaming, and Content-Based Retrieval
description: The graphics and media APIs applications use to reach the hardware, how audio and video travel over networks with RTP and adaptive streaming, how playback stays synchronised, and how multimedia databases search by content rather than by keyword.
date: 2023-06-07
draft: false
slug: /computer-graphics/architecture
tags:
  - Computer Graphics
  - Multimedia
---

Between a multimedia application and the screen sits a stack of infrastructure that is invisible when it works: the APIs that talk to the graphics hardware, the protocols that carry video across the internet, the timing machinery that keeps audio matched to picture, and the databases that let you find an image by what it looks like rather than by a caption someone typed. This post covers all four.

It follows on from [audio and video](/citadel/computer-graphics/audio-video).

## APIs to the hardware

An operating system provides core multimedia support — audio, video, and graphics APIs, codec management, and device handling. Windows, for example, exposes a **Waveform Audio API** for basic recording and playback, and the **DirectShow** (older) and **Media Foundation** (newer) frameworks for full audio and video pipelines: capture, decoding, processing, rendering.

For high-performance rendering, applications go through a graphics library that gives direct access to the GPU:

- **DirectX** — Microsoft's family of multimedia APIs, aimed especially at games, on Windows and Xbox. Components include Direct3D (3D), Direct2D (2D), XAudio2 (audio, replacing the older DirectSound), and DirectInput. It is tightly coupled to the hardware and fast.
- **OpenGL** — a cross-platform, open standard for 2D and 3D vector graphics, managed by the Khronos Group. Used in CAD, virtual reality, scientific visualisation, flight simulation, and games across Windows, macOS, Linux, and mobile.

The trade-off is the usual one: a platform-specific API like DirectX can exploit the hardware fully and evolve with one vendor's roadmap; a cross-platform API like OpenGL runs everywhere at the cost of a lowest-common-denominator feature set and slower standardisation.

## Media over networks

Many multimedia experiences involve data crossing a network.

**Videoconferencing** is real-time two-way audio and video between participants in different places. It requires capture, compression (H.264 or H.265 for video, AAC or Opus for audio), transport (usually RTP), and decompression for playback. The hard parts are **latency** (delay), **jitter** (variation in packet arrival time), and limited bandwidth.

**Video on demand** lets users pick content to watch whenever they want, rather than at a broadcast time. It relies on large libraries of compressed video on servers, streaming protocols, and [content delivery networks](/citadel/interview/cdn) that place copies near users to cut latency. See [Netflix](/citadel/system-design/netflix) for a worked case.

### RTP and streaming

The **Real-Time Transport Protocol** delivers audio and video over IP. It provides payload-type identification, **sequence numbering** so the receiver can reorder packets, and **timestamping** for synchronisation. It usually runs over UDP (see [the transport layer](/citadel/computer-networks/transport-layer)) and pairs with **RTCP**, which carries control messages and quality-of-service feedback.

**Streaming** transmits media as a continuous flow so playback can start while the rest is still arriving — essential for on-demand and live content, where downloading the whole file first is not an option. **HLS** (HTTP Live Streaming, from Apple) and **DASH** (Dynamic Adaptive Streaming over HTTP) both run over ordinary [HTTP/HTTPS](/citadel/interview/https) and adapt the video quality to the viewer's current network conditions.

### Frameworks

Higher-level frameworks bundle these pieces:

- **Windows Media Framework** — Media Foundation at the core, plus codecs, players, and capture, editing, and streaming APIs.
- **QuickTime Architecture** — Apple's framework, built around the `.mov` container; its formats remain influential in professional video even though the Windows player is discontinued.
- **Ogg Framework** — open-source and patent-free: the Ogg container, Vorbis (lossy audio), Theora (lossy video), and Opus (a versatile audio codec used in WebRTC and streaming).

## Synchronisation

In multimedia, timing is a first-class concern. **Temporal relationships** define how media elements relate over time — audio should start when its video clip does, an animation should trigger after a button click. **Synchronisation** is maintaining those relationships during playback. The classic example is **lip-sync**: spoken audio matched to the speaker's lip movements on screen.

It is hard because the audio and video streams take different processing paths with different delays, network jitter perturbs arrival times, and device clocks drift. The techniques: **timestamps** (from RTP) to mark when each unit should play, **buffering** to absorb jitter, and feedback mechanisms that nudge playback rates to keep the streams aligned.

## Multimedia databases

As collections of images, audio, and video grow, finding a specific item becomes the problem.

Traditionally you search by **textual metadata** — keywords, tags, captions. That has limits: descriptions are **subjective** (they vary by who wrote them), **incomplete** (a few words cannot capture a rich image), **labour-intensive** to produce at scale, and **language-dependent**.

**Content-based storage and retrieval** (CBSR; also called content-based image retrieval, CBIR, for images) searches the actual content instead:

1. **Feature extraction** — the system automatically computes low-level features (colours, textures, shapes for images; pitch, rhythm, timbre for audio) and represents them as numerical vectors, or "signatures".
2. **Indexing** — the signatures are stored and [indexed](/citadel/dbms/relational-data-model) for fast search.
3. **Querying** — the user supplies a query: an example image, a sketch, an audio snippet, or "find items similar to this one".
4. **Similarity matching** — the system compares the query's features against the database and returns the closest matches by a similarity metric.

**Image features:**

- **Colour** — colour histograms (the distribution of colours), dominant colours, and colour layout (their spatial arrangement).
- **Texture** — the visual pattern and surface feel of a region, analysed with Gabor filters or co-occurrence matrices.
- **Shape** — geometric properties of objects: contours, area, perimeter, moments. Shape retrieval is harder because it needs the object segmented from the background first — see [segmentation](/citadel/computer-graphics/image-processing).

**Audio features:** a first step is **speech/music discrimination**, since different features matter for each. For music: pitch, melody contour, rhythm (tempo, beat), harmony, timbre. For speech: speech rate, intonation, phonetic content.

CBSR powers reverse image search (Google Images), music identification apps (Shazam), and search over large medical image archives.

## The one idea to keep

Multimedia architecture is layers of indirection that each solve one problem: APIs so an application need not know the GPU, RTP and adaptive streaming so media survives an unreliable network, timestamps and buffering so it stays in sync on arrival, and content-based retrieval so a collection is searchable by what the media contains rather than by what someone remembered to write about it.
