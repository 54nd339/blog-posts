---
title: Engineering YouTube - The Transcoding Pipeline and Custom VCU Hardware
description: More than 500 hours of video are uploaded to YouTube every minute. How the ingestion-to-streaming pipeline is structured, the custom Video Coding Unit hardware that made transcoding 20-33x more efficient, and how live streaming differs.
date: 2025-07-18
draft: false
slug: /system-design/youtube
tags:
  - System Design
  - Case Study
  - Streaming
---

More than 500 hours of video are uploaded to YouTube every minute, on average. Every one of those hours has to be converted into a spread of formats and resolutions, stored, and then served to billions of viewers with the quality tuned to each viewer's connection. The pipeline that does this is standard in shape; the interesting parts are where YouTube had to leave the standard approach — most of all, building its own transcoding silicon.

![The YouTube pipeline: an upload goes to object storage, its metadata to a database and cache, then through an asynchronous transcoding pipeline whose completion notification updates the metadata; playback requests are served from a CDN backed by that object storage](../images/youtube.png "Upload, asynchronous transcode, and CDN-fronted streaming.")

## The pipeline

**Upload.**

1. A user submits the video file and its metadata (title, description, tags).
2. The raw file goes to durable object storage (S3 or Google Cloud Storage).
3. The metadata is written to a primary database and to a cache for fast reads.
4. The raw file enters an **asynchronous transcoding pipeline** — the resource-heavy step of producing formats (MP4, WebM), resolutions (480p through 4K), and bitrates for device and network compatibility.
5. The transcoded versions are written back to object storage.
6. A completion notification (success or failure) is published to a message queue.
7. A "transcoding status handler" consumes that notification and updates the metadata database and cache with the status and file locations.

**Streaming.**

8. A viewer's play request is routed to a CDN.
9. On a cache hit, the edge server returns the video segments directly. On a miss, the CDN pulls them from object storage and caches them for the region. Playback uses adaptive bitrate streaming (**HLS** or **DASH**): the video is cut into a few-second chunks, and the client player picks the quality of each chunk from current network conditions.

## The VCU

Transcoding is YouTube's mission-critical cost centre. When upload and watch volumes surged — notably during the COVID-19 pandemic — software encoding was too slow and too expensive at this scale. So YouTube built dedicated hardware:

- **Video Coding Units (VCUs)** — custom accelerators for warehouse-scale video processing, in the same spirit as GPUs for graphics or TPUs for machine learning.
- **Cluster structure** — VCUs run in clusters: each server holds several accelerator trays, each tray several VCU cards, each card the encoders, decoders, and processing blocks. Raw video goes in, encoded versions come out and land in cloud storage.
- **Result** — a 20–33x improvement in compute efficiency over YouTube's already-optimised software encoder, documented in the 2021 paper *Warehouse-scale video acceleration: co-design and deployment in the wild*.

## Live streaming

Live adds the constraint that everything happens now:

1. The streamer's camera and encoder start the broadcast.
2. The stream is sent to the nearest **point-of-presence (PoP)** server, which acts as the ingestion point and keeps the upload stable.
3. The incoming stream is transcoded in real time into multiple resolutions and bitrates, and simultaneously cut into few-second segments.
4. Segments are packaged for adaptive streaming (HLS or MPEG-DASH), with manifest files listing the available streams and segments.
5. Manifests and chunks are pushed out and cached across the global CDN.
6. Viewers' players fetch the manifest from a nearby edge and then request segments, adapting quality as they go.
7. Optionally, the stream is recorded to object storage for video-on-demand replay afterward.

## The takeaway

The pipeline shape — object storage, async transcode, message-queue notification, CDN-fronted delivery — is what you'd design for any large video platform, and the sibling post on [video streaming](/citadel/system-design/video-streaming) covers it generically. What's specific to YouTube's scale is where a general design stops paying off: transcoding got expensive enough that custom hardware beat a software fleet by more than an order of magnitude. See [CDNs](/citadel/interview/cdn) for the delivery layer and [Netflix](/citadel/system-design/netflix) for the same problem solved with an in-house CDN instead of custom silicon.
