---
title: Goals
description: A running list of the build-it-from-scratch projects I keep meaning to start - a language, an operating system, a browser, a few apps - and the one part of each that is the real work.
date: 2023-07-03
draft: false
slug: /miscs/goals
tags:
  - Notes
  - Projects
---

I keep a list of things I want to build from scratch. Not because the world needs another one of any of them, but because building the small, ugly version of something is the fastest way I know to actually understand it. The pattern is always the same: start with a subset that barely works, lean on existing libraries for the parts that are not the point, and grow it. What follows is the list, and for each item, the part that is the actual work.

## A programming language

The pipeline is well trodden: design the grammar and semantics, write a **lexer** to turn source into tokens, a **parser** to build an abstract syntax tree, a **code generator** that lowers the tree to machine code or an intermediate form, and — if the language needs it — a runtime for garbage collection, exceptions, and concurrency. Lexer and parser generators do the mechanical parts.

The real work is the design. Tokens and trees are a solved problem; deciding what the language *is* — what its constructs mean, what is easy and what is impossible — is the part with no generator for it. See [compilation models](/citadel/coding-languages/coding-langs) and the stages in [lexical](/citadel/compiler-design/lexical-analysis), [syntax](/citadel/compiler-design/syntax-analysis), and [code generation](/citadel/compiler-design/code-genration).

## An operating system

Design the architecture — kernel structure, the system-call interface, the driver model — then write the kernel, the system calls, device drivers, a bootloader, and enough of a linker and assembler to produce a binary that boots. Start with a subset: get something to print to the screen, then schedule two tasks, then handle an interrupt.

The real work is that there is no floor to stand on. Every abstraction a normal program takes for granted, you are building. It demands comfort with low-level code and [computer architecture](/citadel/operating-system/os) that I do not fully have yet, which is the point of doing it.

## A web browser

Pick a rendering engine to start (WebKit, Blink, Gecko), build the surrounding UI — window, toolbar, bookmarks, history — implement the networking stack ([HTTP and HTTPS](/citadel/interview/https), cookies, [caching](/citadel/interview/caching)), add a JavaScript engine, and support the [DOM](/citadel/coding-languages/html) and [CSS](/citadel/coding-languages/css) enough to lay out and paint a page.

The real work is the surface area. A browser is a dozen hard projects — a parser, a layout engine, a JS VM, a network stack, a security model — wearing a trench coat. See [how browsers work](/citadel/system-design/browsers).

## A file-format converter

The most tractable one. Read the specifications for the source and target formats, look at real sample files, write code to parse the source and extract the data, write code to emit the target, and test against a pile of inputs. A CLI or GUI on top if it earns one.

The real work is entirely in the formats. The conversion logic is straightforward once you genuinely understand both ends; the effort is reading specs carefully and handling the files that do not match them.

## A map app

Choose a mapping platform (Google Maps, OpenStreetMap, Mapbox), build the UI — map view, search, navigation, markers — wire up the interactions (pan, zoom, API calls for tiles and places), and manage the data: current location, saved places, routes. Offline tiles, AR, and live tracking are later.

The real work is the rendering and the geospatial data — projections, tile pyramids, coordinate systems. See [map rendering](/citadel/system-design/map-rendering).

## A video-chat app

Start on a real-time platform (WebRTC, Twilio, Agora), build the UI for the video view and call and participant management, implement the connection setup and the sending and receiving of audio and video streams, and store the boring state — call history, profile, settings. Screen sharing, recording, and text chat come after.

The real work is the real-time media path: establishing peer connections through NATs, and keeping latency low enough that a conversation feels like one. See [video streaming](/citadel/system-design/video-streaming).

## A Desmos clone

Study how Desmos handles expressions, graphing, and interaction. Build the UI — calculator panel, graph canvas, input, zoom, pan — then the functionality: parse and evaluate mathematical expressions, and render the graphs. Underneath it needs a math engine for both symbolic and numeric work.

The real work is the math engine — parsing mathematical notation, evaluating it fast enough to graph interactively, and doing the symbolic manipulation. See [computational science](/citadel/maths/numerical-analysis).

## What the list is really for

Every one of these is "a complex and time-consuming task", and every one starts the same way — build the smallest thing that runs, then add. The value is not the finished artifact; it is that after building even a broken version, I stop treating the real one as magic. That is the whole goal.
