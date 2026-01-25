---
title: Web Page Internals - Understanding HTML, DOM, BOM, JSON, and Their Dynamic Dance
description: Ever wondered how web pages come alive? We dive into HTML as the structure, the DOM as its programmable interface, DOM queries & manipulation, the BOM for browser interaction, and JSON as the data workhorse, plus how they all interact in the browser.
date: 2024-08-11
draft: true
slug: /pensieve/html
tags:
  - languages
  - interview
---

Hey everyone, and welcome back to the blog! When we browse the internet, clicking links, filling forms, and interacting with dynamic content, it all seems so fluid and intuitive. From bustling e-commerce sites to interactive dashboards, the web is a rich, responsive experience. But what are the fundamental building blocks that make this possible?

Today, we're going to pull back the curtain on some core technologies that power every webpage you visit: **HTML** (the structure), the **DOM** (its programmable representation), **DOM queries and manipulation** (how we interact with it), the **BOM** (how we talk to the browser), and **JSON** (the data format that glues much of it together). Understanding these is key for any aspiring web developer or system designer.

## A. HTML: The Skeleton of the Web (A Foundational Recap) 뼈대

We've touched on **HTML (HyperText Markup Language)** before. It's the standard markup language used to create and define the *structure and content* of web pages and web applications. HTTP is the protocol for fetching resources such as HTML documents, making HTML a cornerstone of web data exchange. When your browser receives an HTML document, its rendering engine parses this HTML. Interestingly, from a programming paradigm perspective, HTML can be seen as a **declarative language** – you declare the structure you want, and the browser figures out how to render it.

The key output of this HTML parsing process is the Document Object Model (DOM).

## B. The DOM (Document Object Model): HTML's Living Representation
Once the browser gets the HTML, it builds the **Document Object Model (DOM)**.
* **What is the DOM?**
    The DOM is a **programming interface for web documents (HTML and XML)**. It represents the page so that programs and scripts (most commonly JavaScript) can dynamically access and change the document's structure, style, and content. The DOM is an object representation of the HTML document, structured as a **tree of objects** where each HTML element, attribute, and piece of text becomes a node in this tree.
* **Why is the DOM Important?**
    The DOM is absolutely crucial because it provides a standardized way for code to interact with and modify a webpage in real-time *after* it has been loaded. Without the DOM, web pages would be static and unchangeable. It's essentially the bridge that allows JavaScript to make web pages dynamic and interactive. The DOM API is designed to be language-agnostic, meaning its structure and how it's manipulated can be accessed by various programming languages, though JavaScript is by far the most common language used for DOM manipulation in web browsers.

### DOM Queries: Finding Your Way Around the Tree ️
To manipulate a part of the webpage, you first need to select or "query" the specific DOM element(s) you're interested in. JavaScript provides several powerful methods for this:
* **`document.getElementById('someId')`:** Selects a single element that has the specified `id` attribute. Since IDs must be unique within a page, this is a very direct way to target an element.
* **`document.getElementsByTagName('p')`:** Returns a live `HTMLCollection` (an array-like object) of all elements with the given tag name (e.g., all `<p>` elements).
* **`document.getElementsByClassName('my-class')`:** Returns a live `HTMLCollection` of all elements that have the specified class name.
* **`document.querySelector('div .item > span')`:** A very versatile method that uses CSS-style selectors to find the *first* element in the document that matches the specified selector string. This allows for complex selections.
* **`document.querySelectorAll('div .item')`:** Similar to `querySelector`, but returns a *static* `NodeList` (another array-like object) representing all elements in the document that match the specified CSS selector string.

### DOM Manipulation: Changing the Page on the Fly ✏️
Once you've selected a DOM element (or elements) using query methods, you can use JavaScript to manipulate its properties, content, and structure:
* **Changing HTML Content & Text:**
    * `element.innerHTML = "<strong>New</strong> content!";` (Replaces the HTML content *inside* the element. Be cautious with user-supplied data due to XSS risks).
    * `element.textContent = "New safe text!";` (Replaces only the text content, treating any HTML markup as literal text, which is safer).
    * `element.innerText` (Similar to `textContent` but with some differences in how it handles hidden elements and spacing).
* **Changing HTML Attributes:**
    * `element.setAttribute('src', 'new-image.jpg');` (Sets any attribute).
    * `element.getAttribute('href');` (Gets the value of an attribute).
    * `element.removeAttribute('disabled');` (Removes an attribute).
    * Direct property access for common attributes: `element.id = 'newId';`, `element.src = 'another-image.png';`, `element.className = 'new-class another-class';`.
* **Changing CSS Styles:**
    * `element.style.color = 'blue';`
    * `element.style.fontSize = '16px';` (Note: CSS properties with hyphens like `font-size` become camelCased in JavaScript).
    * `element.classList.add('active');` (Recommended for managing multiple style changes via CSS classes).
    * `element.classList.remove('hidden');`
    * `element.classList.toggle('highlight');`
* **Creating and Adding/Removing Elements:**
    * `let newParagraph = document.createElement('p');`
    * `newParagraph.textContent = "This is a new paragraph.";`
    * `parentElement.appendChild(newParagraph);` (Adds the new element as the last child of `parentElement`).
    * `parentElement.insertBefore(newParagraph, existingChildElement);` (Inserts before a specific child).
    * `parentElement.removeChild(childElementToRemove);`
    * `element.remove();` (A simpler way to remove an element).
* **Working with Events:**
    * `buttonElement.addEventListener('click', function(event) { console.log('Button clicked!', event); });` (Allows JavaScript to respond to user interactions like clicks, mouse movements, key presses, etc.).

## C. The BOM (Browser Object Model): Interacting with the Browser Itself 윈도우

While the DOM represents the document (the webpage content itself), the **Browser Object Model (BOM)** provides an interface for JavaScript to interact with the browser window and features outside the specific content of the page. Unlike the DOM, which is well-standardized by the W3C, the BOM is less standardized and has historically had more browser-specific implementations, though many common features are now widely supported.

* **What is the BOM?**
    The BOM is a browser-specific convention referring to all objects exposed by the web browser to JavaScript. It allows scripts to "talk to" and control aspects of the browser window. The global `window` object is the root of the BOM and also acts as the global object for JavaScript running in the browser environment.
* **Key BOM Objects and Their Uses:**
    * **`window` object:** The top-level object in the BOM, representing the browser window or tab containing the DOM document.
        * *Properties:* `window.innerHeight` and `window.innerWidth` (current viewport dimensions), `window.document` (which *is* the DOM's `document` object – the entry point to the page's content), `window.location` (current URL info), `window.navigator` (browser info), `window.screen` (screen info), `window.history` (browser history).
        * *Methods:* `window.alert()`, `window.confirm()`, `window.prompt()` (for simple user dialogs), `window.open()` (to open new windows/tabs), `window.close()` (to close windows opened by script), `setTimeout()`, `setInterval()` (for timed execution of code), `window.scrollTo()`, `window.scrollBy()`.
    * **`navigator` object (`window.navigator`):** Provides information about the browser itself and the system it's running on.
        * Examples: `navigator.userAgent` (browser identification string), `navigator.language` (preferred language), `navigator.onLine` (network connection status), `navigator.geolocation` (for accessing user's location, with permission), `navigator.mediaDevices` (for accessing camera/microphone).
    * **`screen` object (`window.screen`):** Contains information about the user's display screen.
        * Examples: `screen.width`, `screen.height` (total screen dimensions), `screen.availWidth`, `screen.availHeight` (available screen space), `screen.colorDepth`, `screen.pixelDepth`.
    * **`location` object (`window.location`):** Provides information about the current URL of the page and allows for navigation.
        * Examples: `location.href` (get or set the full URL), `location.hostname`, `location.pathname`, `location.search` (query parameters), `location.hash` (fragment identifier), `location.reload()` (reload the page), `location.assign('https://new.com')` (navigate to a new page).
    * **`history` object (`window.history`):** Allows interaction with the browser's session history (the pages visited in the current tab/window within the same origin).
        * Examples: `history.back()` (go to previous page), `history.forward()` (go to next page), `history.go(n)` (go n pages forward/backward), `history.pushState()` and `history.replaceState()` (for manipulating browser history without full page reloads, used in SPAs).

## D. JSON (JavaScript Object Notation): The Lingua Franca for Web Data
When web applications need to send or receive data from a server (e.g., via AJAX or the Fetch API), they need a common, lightweight format for that data. **JSON** has emerged as the de facto standard for this.

* **What is JSON?**
    **JSON (JavaScript Object Notation)** is a lightweight data-interchange format. It is designed to be easy for humans to read and write, and equally easy for machines to parse and generate. While its syntax is a subset of JavaScript object literal notation, JSON is language-independent, with parsers and generators available for nearly all programming languages.
* **Structure:** JSON is built on two fundamental structures:
    * **Objects:** A collection of key/value pairs, enclosed in curly braces `{}`. Keys *must* be strings (enclosed in double quotes), and values can be strings (in double quotes), numbers, booleans (`true` or `false`), arrays, other JSON objects, or `null`.
        ```json
        {
          "name": "Aditi Sharma",
          "city": "Bengaluru",
          "isDeveloper": true,
          "skills": ["Java", "Python", "System Design"],
          "experience": null
        }
        ```
    * **Arrays:** An ordered list of values, enclosed in square brackets `[]`. Values can be of any valid JSON type.
        ```json
        ["JavaScript", "HTML", "CSS", 2025, true]
        ```
* **Why it's Popular for APIs and Web Communication:**
    * **Native JavaScript Support:** JavaScript has built-in methods to easily parse JSON strings into JavaScript objects (`JSON.parse()`) and to convert JavaScript objects/values into JSON strings (`JSON.stringify()`).
    * **Compact & Readable:** Generally more compact and easier for humans to read compared to older formats like XML for many common data interchange use cases. This efficiency is also noted when contrasting it with highly optimized binary formats like Protocol Buffers, where Protobuf is even smaller and faster for specific scenarios like gRPC.
    * **Widely Used:** The de facto standard for transmitting data between a server and a web application in RESTful APIs and asynchronous requests. Also used extensively for configuration files and NoSQL document databases (like MongoDB, which uses BSON, a binary extension of JSON).

## E. "Everything Around It": The Broader Web Ecosystem

These technologies form a cohesive ecosystem that brings modern web pages to life:

* **Browser Rendering Pipeline Recap:** As a reminder from our previous discussions, the browser fetches an **HTML** document (often mentioned as a resource fetched by HTTP), parses it into the **DOM** tree. Linked **CSS** files (static assets often served from CDNs) are parsed into the **CSSOM**. These two trees are then combined to create the **Render Tree**. The browser subsequently performs **Layout (Reflow)** to calculate the geometry of each visible element and finally **Paints** the pixels to the screen.
* **JavaScript's Central Role:** JavaScript is the primary scripting language that acts as the "engine" of interactivity. It uses the DOM API to query and manipulate the page structure and content, and the BOM API to interact with browser features.
* **AJAX (Asynchronous JavaScript and XML) & Fetch API:**
    * AJAX is a set of web development techniques used to create asynchronous web applications. With AJAX, web applications can send and retrieve data from a server asynchronously (in the background) without interfering with the display and behavior of the existing page, allowing for partial page updates.
    * While "XML" is in its name, **JSON** is now the predominant data format used with AJAX.
    * The modern **Fetch API** provides a more powerful and flexible interface for making network requests (the successor to the older `XMLHttpRequest` object traditionally used for AJAX).
* **Web APIs (Browser-provided APIs):** The DOM and BOM are essentially collections of APIs provided by the browser environment. Browsers expose a vast array of other Web APIs to JavaScript, enabling richer application functionality, such as:
    * `Geolocation API`: For accessing user's location (with permission).
    * `Web Storage API`: `localStorage` and `sessionStorage` for client-side data storage.
    * `Workspace API` / `XMLHttpRequest`: For making network requests.
    * `Canvas API` / `WebGL`: For 2D and 3D graphics rendering.
    * `WebSockets API`: For real-time, bidirectional communication.
    * And many more for audio, video, web workers, service workers, etc.

## Key Takeaways

* **HTML** is the foundational markup language that defines the structure and content of web pages.
* The browser parses HTML into a tree-like **DOM (Document Object Model)**, which serves as a live, programmable interface to the page's content.
* **JavaScript** uses **DOM Query** methods to select elements and **DOM Manipulation** methods to dynamically change a page's content, structure, and style, making web pages interactive.
* The **BOM (Browser Object Model)**, with the `window` object at its core, allows JavaScript to interact with the browser window itself (e.g., managing navigation, screen properties, timers, alerts).
* **JSON** has become the standard lightweight format for data interchange between web clients and servers, easily parsed and generated by JavaScript.
* These core technologies, along with CSS and the browser's rendering engine, work in concert to create the rich, dynamic, and interactive web experiences we rely on every day.

Understanding how these fundamental pieces of the web puzzle fit together is crucial for anyone involved in web development or designing systems that interact with web frontends. They are the very fabric of the interactive web!