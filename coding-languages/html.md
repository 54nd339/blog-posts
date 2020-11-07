---
title: HTML, the DOM, the BOM, and JSON - How a Page Becomes Programmable
description: A browser parses HTML into the DOM - a tree of objects that JavaScript can query and change, which is what makes a page interactive. The DOM query and manipulation APIs, the BOM for talking to the browser window itself, and JSON as the format that moves data between client and server.
date: 2020-11-07
draft: false
slug: /coding-languages/html
tags:
  - Programming Languages
  - Web
  - JavaScript
---

A web page is static text until something makes it move. That something is the **DOM** — the browser's object model of the page — which [JavaScript](/citadel/coding-languages/javascript) reads and rewrites in response to clicks, input, and network results. This post covers HTML as the structure, the DOM as its programmable form, the query and manipulation APIs, the **BOM** for the browser window around the page, and **JSON** as the data format that ties client and server together.

## HTML: the structure

**HTML (HyperText Markup Language)** defines the structure and content of a page. [HTTP](/citadel/interview/http) fetches the HTML document; the browser's rendering engine parses it. HTML is **declarative** — you state the structure you want and the browser decides how to render it. The output of parsing is the DOM.

## The DOM: HTML as objects

Once the browser has the HTML, it builds the **Document Object Model** — a **tree of objects**, one node per element, attribute, and text run. The DOM is a **programming interface**: scripts can read and change the document's structure, style, and content after it has loaded. That is the bridge that turns a static page dynamic. The API is language-agnostic in principle; in browsers it is used almost entirely from JavaScript.

### DOM queries: selecting elements

To change part of a page you first select the node(s):

- `document.getElementById('someId')` — the one element with that `id`.
- `document.getElementsByTagName('p')` — a live `HTMLCollection` of all `<p>` elements.
- `document.getElementsByClassName('item')` — a live `HTMLCollection` by class.
- `document.querySelector('div .item > span')` — the first element matching a CSS selector.
- `document.querySelectorAll('div .item')` — a static `NodeList` of all matches.

### DOM manipulation: changing the page

- **Content:** `element.innerHTML = "<strong>New</strong>"` (parses HTML — an XSS risk with untrusted data); `element.textContent = "safe text"` (treats markup as literal text).
- **Attributes:** `setAttribute('src', 'x.jpg')`, `getAttribute('href')`, `removeAttribute('disabled')`, or direct properties (`element.id = 'newId'`).
- **Styles:** `element.style.color = 'blue'` (hyphenated CSS names become camelCase: `fontSize`); `element.classList.add('active')`, `.remove('hidden')`, `.toggle('highlight')` — the preferred way, since it keeps styling in [CSS](/citadel/coding-languages/css).
- **Structure:** `document.createElement('p')`, then `parent.appendChild(node)`, `parent.insertBefore(node, ref)`, `parent.removeChild(child)`, or `element.remove()`.
- **Events:** `button.addEventListener('click', e => { ... })` to respond to interaction.

## The BOM: talking to the browser

The DOM represents the *document*. The **Browser Object Model** represents the *browser window* and things outside the page content. It is less standardised than the DOM, though the common parts are widely supported. The global `window` object is its root, and also JavaScript's global object in the browser.

- **`window`** — the browser window or tab.
  - Properties: `innerHeight` / `innerWidth` (viewport size), `document` (the DOM entry point), `location`, `navigator`, `screen`, `history`.
  - Methods: `alert()`, `confirm()`, `prompt()`; `open()`, `close()`; `setTimeout()`, `setInterval()`; `scrollTo()`, `scrollBy()`.
- **`navigator`** — the browser and system: `userAgent`, `language`, `onLine`, `geolocation`, `mediaDevices`.
- **`screen`** — the display: `width`, `height`, `availWidth`, `availHeight`, `colorDepth`.
- **`location`** — the current URL, and navigation: `href` (read or set), `hostname`, `pathname`, `search`, `hash`, `reload()`, `assign(url)`.
- **`history`** — session history: `back()`, `forward()`, `go(n)`, and `pushState()` / `replaceState()` for changing the URL without a full page load (how SPAs do routing).

## JSON: moving data

When a page sends or receives data, it needs a lightweight, language-independent format. **JSON (JavaScript Object Notation)** is the de facto standard. Its syntax is a subset of JavaScript object literals, but parsers exist for nearly every language.

Two structures:

- **Objects** — key/value pairs in `{}`. Keys are double-quoted strings; values are strings, numbers, booleans, arrays, objects, or `null`.

  ```json
  {
    "name": "widget",
    "region": "eu-west",
    "active": true,
    "tags": ["storage", "beta"],
    "retired": null
  }
  ```

- **Arrays** — ordered lists in `[]`, of any JSON values.

  ```json
  ["html", "css", "javascript", 2025, true]
  ```

Why it took over web APIs:

- **Native in JavaScript** — `JSON.parse()` string to object, `JSON.stringify()` object to string.
- **Compact and readable** — smaller and easier to skim than XML for typical payloads. Binary formats like Protocol Buffers are smaller and faster still for specific cases like [gRPC](/citadel/interview/grpc).
- **Ubiquitous** — the standard for REST APIs and AJAX, and for config files and document databases (MongoDB stores BSON, a binary JSON).

## How it fits together

- **Rendering pipeline:** the browser fetches **HTML**, parses it into the **DOM** tree; linked **CSS** parses into the **CSSOM**; the two combine into the **render tree**; the browser does **layout** to compute geometry, then **paints** pixels. See [frontend performance](/citadel/interview/fe-performance).
- **JavaScript** is the engine of interactivity: it uses the DOM API to change the page and the BOM API to reach browser features.
- **AJAX and the Fetch API:** AJAX is the technique of exchanging data with a server in the background for partial page updates. "XML" is in the name, but JSON is what is actually used. The modern **Fetch API** is the current interface, replacing `XMLHttpRequest`.
- **Web APIs** — the DOM and BOM are themselves browser-provided APIs; browsers expose many more: Geolocation, Web Storage (`localStorage`, `sessionStorage`), Fetch, Canvas and WebGL, WebSockets, plus audio, video, web workers, and service workers.

## Key takeaways

- **HTML** declares structure; the browser parses it into the **DOM**, a tree of objects.
- **DOM queries** (`getElementById`, `querySelector`) select nodes; **DOM manipulation** (`textContent`, `classList`, `createElement`, `addEventListener`) changes them — this is what makes a page interactive.
- The **BOM**, rooted at `window`, is how JavaScript reaches the browser itself: navigation, screen, history, timers, dialogs.
- **JSON** — objects and arrays, parsed and generated natively by JavaScript — is the standard format for client/server data.
