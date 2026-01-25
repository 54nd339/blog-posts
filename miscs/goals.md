---
title: Goals
description: Projects I want to do before my Bachelors
date: 2023-01-19
draft: true
slug: /pensieve/goals
tags:
  - OS
  - Compiler
---
## How to make a new programming language using C language?

Creating a new programming language using C as the implementation language for the compiler is a complex and time-consuming task. Here are the general steps you would need to take:

1. Design the syntax and semantics of the new language. This includes creating a grammar for the language, which defines the structure of valid programs, and determining the behavior of the language's constructs.
2. Write a lexer in C to tokenize the source code. The lexer reads the source code and converts it into a stream of tokens, which are the basic building blocks of the language.
3. Write a parser in C to analyze the structure of the program represented by the stream of tokens. The parser creates an abstract syntax tree (AST) which represents the structure of the program.
4. Implement a code generation component that takes the AST as input and produces machine code or intermediate code as output.
5. Develop a runtime system, if necessary, to support features such as garbage collection, exception handling, and concurrency.
6. Test the compiler thoroughly to ensure that it produces correct code and can handle a variety of inputs.

It's worth noting that creating a new language is a complex and time-consuming task, and it's a good idea to start with a small subset of the language and gradually build on it. Additionally, it's a good idea to use existing libraries and tools, such as lexer and parser generators, to help with the development.

## How to make a OS using C language?

Creating an operating system (OS) using C as the implementation language is a challenging task that requires a deep understanding of both C programming and operating system design. Here are the general steps you would need to take:

1. Design the architecture of the operating system. This includes determining the structure of the kernel, the system calls, and the device drivers.
2. Write the kernel in C. The kernel is the central part of the operating system that controls the resources of the computer, including the CPU, memory, and devices.
3. Implement system calls in C. System calls provide an interface for programs to request services from the operating system.
4. Develop device drivers in C. Device drivers are software modules that allow the operating system to communicate with hardware devices.
5. Create a bootloader in C. The bootloader is a small program that is executed when the computer starts up and is responsible for loading the operating system into memory.
6. Write a linker and an assembler in C to generate the final executable binary file.
7. Test the operating system thoroughly to ensure that it can handle a variety of inputs and configurations.

It's worth noting that creating a new OS is a very complex task and it's a good idea to start with a small subset of the operating system and gradually build on it. Additionally, it's a good idea to use existing libraries and tools, such as bootloaders and file systems, to help with the development. It's also important to have a strong knowledge of low-level programming and computer architecture to create an OS from scratch.

## How to make a web browser?

Creating a web browser is a complex task that requires knowledge of several different technologies, including web standards, networking, and user interface design. Here are the general steps you would need to take:

1. Choose a rendering engine. A rendering engine is the software component that is responsible for displaying web content. Popular choices include WebKit, Blink and Gecko.
2. Design the user interface. This includes creating the layout of the browser window, designing the toolbars and buttons, and implementing features such as bookmarks and history.
3. Implement the networking stack. This includes writing code to handle HTTP and HTTPS requests, cookies, and caching.
4. Implement JavaScript support. This includes writing a JavaScript engine that can execute scripts and handle events.
5. Implement the Document Object Model (DOM) and Cascading Style Sheets (CSS) support. These technologies are used to create and manipulate the structure and layout of web pages.
6. Test the browser thoroughly to ensure that it can handle a variety of web pages and that the output is correct.

It's worth noting that creating a web browser is a very complex task and it's a good idea to start with a small subset of the browser and gradually build on it. Additionally, it's a good idea to use existing libraries and tools, such as web rendering engines, to help with the development. It's also important to have a strong knowledge of web standards and web development to create a web browser.

## how to make a file format convertor?

Creating a file format converter is a relatively straightforward task, but it does require knowledge of the specific file formats you are working with. Here are the general steps you would need to take:

1. Understand the source and target file formats. This includes reading the specifications of the file formats and examining sample files to see how the data is structured.
2. Write code to read the data from the source file format. This typically involves using libraries or existing code to parse the file and extract the relevant data.
3. Write code to write the data to the target file format. This typically involves using libraries or existing code to create the new file and insert the data in the correct format.
4. Test the converter thoroughly to ensure that it can handle a variety of inputs and that the output is correct.
5. Optionally, you can add features such as a graphical user interface, or a command-line interface, to make it more user-friendly.

It's worth noting that depending on the complexity of the file format, it could be a simple or a complex task. Additionally, it's a good idea to use existing libraries or tools that handle the file format you are working with.

## how to make a map app?

Creating a map app involves several steps and technologies, including design, development, and data management. Here are the general steps you would need to take:

1. Choose a mapping platform. There are several mapping platforms available such as Google Maps, OpenStreetMap, and Mapbox that provide APIs for adding maps and location-based functionality to your app.
2. Design the user interface. This includes creating the layout of the app, designing the map view, and implementing features such as search, navigation, and markers.
3. Implement the functionality. This includes writing code to handle user interactions, such as panning and zooming, and to access the mapping platform's API to retrieve and display maps, location data, and other information.
4. Implement data management. This includes writing code to store and retrieve data such as user's location, saved places, and routes.
5. Test the app thoroughly to ensure that it can handle a variety of inputs and that the output is correct.
6. Optionally, you can add features such as offline map support, augmented reality, and real-time location tracking.

It's worth noting that creating a map app is a complex task and it's a good idea to start with a small subset of the app and gradually build on it. Additionally, it's a good idea to use existing libraries and tools, such as mapping platform's SDK, to help with the development. It's also important to have a strong knowledge of mobile app development and geospatial data to create a map app.

## how to make a video chat app?

Creating a video chat app involves several steps and technologies, including design, development, and deployment. Here are the general steps you would need to take:

1. Choose a platform. There are several platforms available such as WebRTC, Twilio, and Agora that provide APIs for adding real-time audio and video functionality to your app.
2. Design the user interface. This includes creating the layout of the app, designing the video view, and implementing features such as call initiation, call management and participant management.
3. Implement the functionality. This includes writing code to handle user interactions, such as call initiation, call management and participant management. It also involves code to access the real-time communication platform's API to establish connections, send and receive audio and video streams, and handle other real-time communication related tasks.
4. Implement data management. This includes writing code to store and retrieve data such as call history, user's profile and call settings.
5. Test the app thoroughly to ensure that it can handle a variety of inputs and that the output is correct.
6. Optionally, you can add features such as screen sharing, recording, and text chat.

It's worth noting that creating a video chat app is a complex task and it's a good idea to start with a small subset of the app and gradually build on it. Additionally, it's a good idea to use existing libraries and tools, such as WebRTC, Twilio, and Agora, to help with the development. It's also important to have a strong knowledge of web and mobile app development, real-time communication and security to create a video chat app.

## how to make a desmos clone?

Creating a clone of the popular online graphing calculator Desmos would involve several steps, including design, development, and testing. Here are the general steps you would need to take:

1. Study and understand the Desmos calculator's features and user interface. This includes examining how it handles mathematical expressions, graphing, and user interactions.
2. Design the user interface. This includes creating the layout of the calculator, designing the graph view, and implementing features such as input, zoom, and pan.
3. Implement the functionality. This includes writing code to handle user input, parsing and evaluating mathematical expressions, and generating graphs.
4. Implement the mathematical engine. This includes writing code to perform mathematical operations, such as algebraic and symbolic computations, and numerical calculations.
5. Test the calculator thoroughly to ensure that it can handle a variety of inputs and that the output is correct.
6. Optionally, you can add features such as support for more advanced mathematical functions, interactive graph manipulation, and animations.

It's worth noting that creating a Desmos clone is a complex task and it's a good idea to start with a small subset of the calculator and gradually build on it. Additionally, it's a good idea to use existing libraries and tools, such as mathematical libraries, to help with the development. It's also important to have a strong knowledge of mathematics and web development to create a Desmos clone.
