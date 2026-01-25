---
title: Slack Deconstructed - How Your Messages and Notifications Magically Appear!
description: Ever marveled at Slack's real-time messaging and seemingly intuitive notifications? We dive into the journey of a Slack message, exploring its server architecture, WebSockets, and the complex logic behind deciding when to ping you.
date: 2024-04-26
draft: false
slug: /pensieve/slack
tags:
  - system-design
  - SDE
---

Hey everyone, and welcome back to the blog! If you're part of a modern team, chances are Slack is your digital office – the bustling hub for messages, file sharing, and those all-important GIF battles. It feels so seamless, doesn't it? You type a message, hit send, and almost instantly, it appears for your colleagues, and perhaps a notification pops up on their devices.

But have you ever wondered about the intricate dance of servers and protocols that makes this real-time magic happen? Today, we're pulling back the curtain on how Slack works, looking at the journey of a message and the surprisingly complex logic behind even a "simple" notification.

## The Core Mission: Delivering Messages in Real-Time

At its heart, Slack is all about facilitating instant communication for teams. This means reliably delivering messages to the right people, in the right channels, with minimal delay. Let's follow a message from sender to receiver.

![Slack Architecture](https://d34u8crftukxnk.cloudfront.net/slackpress/prod/sites/7/0_PV_09olld6K1l8jQ.png)

## The Journey of a Slack Message: A Multi-Server Relay

According to technical articles from Slack's own engineering team, delivering a message involves a sophisticated multi-step process through various specialized servers:

1.  **Client Interaction & The API Layer (WebApp):**
    * It all starts when you, the user, type a message in your Slack client (be it desktop, mobile, or web) and hit send.
    * Your client interacts with a **WebApp** server. This WebApp defines the API that Slack clients use to send and receive information.

2.  **Finding the Right Channel's Home (Admin Server):**
    * Once the WebApp receives your message, it needs to know where to send it. This is where the **Admin Server (AS)** comes in.
    * The AS is responsible for finding the correct **Channel Server (CS)** that hosts the specific channel or direct message you're sending to, typically using a channel ID for the lookup.

3.  **Managing Channels at Scale (Channel Server & Consistent Hashing):**
    * Slack handles millions of channels. To manage this immense scale, **Channel Servers (CS)** use techniques like consistent hashing to distribute the load of these channels across many CS instances.
    * The specific CS responsible for your target channel receives the message. This CS is the keeper of that channel's state and message history.

4.  **Pushing Updates to Receivers (Gateway Server & Envoy):**
    * After the Channel Server processes the incoming message (e.g., stores it, updates channel state), it needs to deliver this new message to all connected members of that channel or direct message.
    * The CS pushes the message out through a **Gateway Server (GS)**. These Gateway Servers are strategically deployed in various geographic regions to be closer to users and manage their active connections.
    * Often, a service proxy like **Envoy** is used in this path. Envoy is a popular choice for cloud-native applications to manage network traffic between services.

5.  **Real-Time Delivery (WebSockets):**
    * How do the Slack clients of other users receive the message almost instantly? The answer is **WebSockets**.
    * Message receivers (i.e., other users' Slack clients) maintain a persistent, bi-directional WebSocket connection with their respective regional Gateway Server. This allows the GS to "push" new messages and updates to clients in real-time, eliminating the need for clients to constantly poll the server for new information.

## The Intricacy of a "Simple" Feature: Deciding to Send a Notification

Beyond just delivering messages to an open client, Slack also needs to decide when and how to send you a notification (that little ping or banner). This might seem straightforward, but it's a prime example of how even seemingly simple features in a large-scale application can involve a surprising amount of logic.

The process Slack uses to decide whether to send a notification has been shared as a flowchart, and it's a fantastic illustration of why a simple feature may take much longer to develop than many people think. When a system has a great design, users often don't even notice the underlying complexity because it just feels like the feature is working as intended.

This decision involves checking a multitude of factors, including:
* Is it a thread message, and is the user subscribed?
* Is the channel muted?
* Is the user in Do Not Disturb (DnD) mode? And are there any DnD overrides?
* Is it a channel-wide mention like `@channel`, `@everyone`, or `@here`?
* What are the user's specific notification preferences for that device (e.g., "Everything," "Mentions," or "Nothing")?
* Is it a direct message (DM)?
* Does the message contain a highlight word for the user?
* What is the user's current presence (active/away)?
* Is the user on a mobile device, and have they passed the mobile push timing threshold (to avoid instant mobile notifications if they're active on desktop)?

This intricate logic ensures notifications are relevant and not overly intrusive.

## Key Architectural Principles at Play (Inferred)

From what we know, Slack's architecture likely embraces several key principles:

* **Microservices:** The distinct roles of the WebApp, Admin Server, Channel Server, and Gateway Server strongly suggest a microservices-based approach, allowing for independent scaling and development.
* **Scalability:** Techniques like consistent hashing for Channel Servers and regional deployment of Gateway Servers are vital for handling Slack's massive user base and message volume.
* **Real-time Communication:** WebSockets are the cornerstone for instant message delivery and presence updates.
* **Resilience & Fault Tolerance:** While not explicitly detailed in every source, systems at Slack's scale must be designed for fault tolerance. The question of how Channel Servers might recover using consistent hashing if they go down points to this being a critical design consideration.

## Key Takeaways

* Slack's ability to deliver messages and notifications in real-time to millions of users relies on a sophisticated, distributed backend architecture.
* Key components include WebApp servers for API interactions, Admin Servers for routing, scalable Channel Servers for message handling, and regional Gateway Servers managing WebSocket connections with clients.
* Even seemingly simple features like notifications involve complex decision trees to provide a user-friendly experience.
* Architectural patterns like microservices, consistent hashing, and the use of WebSockets are fundamental to Slack's operation.

The next time you send a message on Slack, take a moment to appreciate the complex journey it undertakes in milliseconds to connect you with your team!
