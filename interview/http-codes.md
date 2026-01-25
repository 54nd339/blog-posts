---
title: HTTP Status Codes - Your Guide to Web Conversations
description: Ever wondered what those three-digit numbers that pop up when you're Browse mean? Let's decode the language of the web with HTTP status codes!
date: 2024-04-04
draft: false
slug: /pensieve/http-codes
tags:
  - interview
  - SDE
---

Hey everyone, and welcome back to the blog! Ever been Browse the web and suddenly hit a page that says "404 Not Found"? Or maybe you've seen a "200 OK" flash by in your developer tools? These three-digit numbers are called HTTP status codes, and they're like the secret handshakes of the internet, letting your browser and the web server communicate effectively. Think of them as quick updates on how your request to a website is going.

Understanding these codes isn't just for tech wizards; it can actually help anyone understand what's happening behind the scenes when they're online. So, let's dive in and make these codes less cryptic and more like friendly signposts on your web journey!

## What's the Deal with HTTP Status Codes?

When your browser wants to get a webpage, it sends a request to the server where the website lives. The server then sends back the requested information along with an HTTP status code. This code is a concise way for the server to tell your browser what happened with its request.

These codes are neatly organized into five main categories, each starting with a different digit. Let's break them down!

![HTTP Status Codes](https://yt3.ggpht.com/lim3q3-pqN6DeEcriI8kKDTfbEx7C8HD3WaVgulOhaJ7-3ybgkOmFxonhac6xuYjEc94ttgxt1CyvN4=s1166-c-fcrop64=1,043f0000fbc0ffff-rw-nd-v1)

### The Five Families of HTTP Status Codes

According to RFC 9110, the response codes for HTTP are divided into these five classes:

* **1xx (Informational):** "Hold on, I'm working on it!" These codes mean the server has received the request and is continuing to process it. You don't often see these directly in your browser.
* **2xx (Success):** "Got it, and here you go!" This is the category you want to see. It means the request was successfully received, understood, and accepted.
* **3xx (Redirection):** "Whoops, not here! Try this new place." These codes indicate that the client needs to take additional action to complete the request, usually by going to a different URL.
* **4xx (Client Error):** "Uh oh, you made a mistake!" This group of codes means there was likely an error in the request made by the client (your browser).
* **5xx (Server Error):** "Oops, my bad!" These codes indicate that the server failed to fulfill a valid request.

Now, let's look at some of the most common codes you might encounter from each family.

### 1xx: Informational – "Just an Update!"

You'll rarely see these directly, but they're part of the conversation.

* **100 Continue:** The server has received the request headers and the client should proceed to send the request body.
* **101 Switching Protocols:** The server is switching protocols as requested by the client.

### 2xx: Success – "All Good!"

These are the happy codes!

* **200 OK:** This is the standard response for successful HTTP requests. The meaning of "success" depends on the HTTP method used:
  * GET: The resource has been fetched and is transmitted in the message body.
  * HEAD: The entity headers for the requested resource are in the message body.
  * POST or PUT: The resource describing the result of the action is transmitted in the message body.
  * TRACE: The message body contains the request message as received by the server.
* **201 Created:** The request has been fulfilled, resulting in the creation of a new resource.
* **202 Accepted:** The request has been accepted for processing, but the processing has not been completed.
* **204 No Content:** The server successfully processed the request but is not returning any content. This is often used for `DELETE` requests.

### 3xx: Redirection – "Head This Way!"

When a resource has moved, these codes point you in the right direction.

* **301 Moved Permanently:** The requested resource has been assigned a new permanent URI and any future references to this resource should use one of the returned URIs. Search engines usually update their links when they see this.
* **302 Found (Previously "Moved Temporarily"):** The server is currently responding to the request with a URI for a different resource but the client should continue to use the original URI for future requests.
* **304 Not Modified:** Indicates that the resource has not been modified since the version specified by the request headers (`If-Modified-Since` or `If-None-Match`). The client can use its cached version.

### 4xx: Client Error – "Looks Like Your Fault!"

These codes usually mean the browser or user did something wrong.

* **400 Bad Request:** The server cannot or will not process the request due to something that is perceived to be a client error (e.g., malformed request syntax).
* **401 Unauthorized:** Although the HTTP standard specifies "unauthorized", semantically this response means "unauthenticated". That is, the client must authenticate itself to get the requested response.
* **403 Forbidden:** The client does not have access rights to the content; that is, it is unauthorized, so the server is refusing to give the requested resource. Unlike 401, the client's identity is known to the server.
* **404 Not Found:** The server cannot find the requested resource. This is the classic "Oops, that page doesn't exist!" error.
* **405 Method Not Allowed:** The request method is known by the server but is not supported by the target resource.
* **408 Request Timeout:** The server would like to shut down this unused connection.
* **429 Too Many Requests:** The user has sent too many requests in a given amount of time ("rate limiting").

### 5xx: Server Error – "It's Not You, It's Me!"

When things go wrong on the website's end.

* **500 Internal Server Error:** A generic error message, given when an unexpected condition was encountered and no more specific message is suitable. This is often a catch-all for server-side bugs.
* **502 Bad Gateway:** The server, while acting as a gateway or proxy, received an invalid response from the upstream server.
* **503 Service Unavailable:** The server is not ready to handle the request. Common causes are a server that is down for maintenance or that is overloaded.
* **504 Gateway Timeout:** The server, while acting as a gateway or proxy, did not get a response in time from the upstream server.

## Key Takeaways

Phew! That's a lot of codes, but here's the gist:

* HTTP status codes are the server's way of communicating the outcome of a client's request.
* They are grouped into five classes (1xx, 2xx, 3xx, 4xx, 5xx) indicating different types of responses.
* Knowing common codes like 200, 301, 404, and 500 can help you diagnose issues or understand what's happening when you browse the web.

These codes are fundamental to how the web works, facilitating the smooth (or sometimes not-so-smooth) flow of information between your computer and the vast network of servers out there.
