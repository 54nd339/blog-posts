---
title: URI, URL, and URN - Names Versus Addresses
description: A URI identifies a resource; a URL is a URI that also says where it is and how to fetch it; a URN is a URI that only names it, permanently and location-independently. The syntax of each and why the distinction matters.
date: 2024-08-24
draft: false
slug: /interview/resource-locator
tags:
  - System Design
  - Interview Prep
  - Web
---

Everyone uses URLs; the terms URI and URN cause confusion because the three overlap. The short version: **URI** is the umbrella, and **URL** and **URN** are both kinds of URI — one an address, one a name.

![Three stacked panels. URL: http :// www.mysite.com : 80 /path/to/mypage.html ?product=camera #SomewhereInDoc, labelled scheme, domain name, port, path, query, anchor. URI (the general form): mailto:Alice@mysite.com with scheme and path, and an ldap:// example with scheme, authority, path, query. URN: urn:isbn:123456 labelled scheme, namespace, namespace-specific string. Arrows note that a URL is a subtype of URI, and a URN is a subtype of URI.](../images/resource-locator.png "URI is the general identifier syntax; URL and URN are both subtypes of it.")

## URI — the identifier

A **Uniform Resource Identifier** is a compact string that identifies a resource — a page, an image, a book, an abstract concept. Its general syntax:

```
scheme:[//authority]path[?query][#fragment]
```

- **scheme** — the protocol or system: `http`, `https`, `ftp`, `mailto`, `file`, `urn`.
- **authority** (optional, after `//`) — host (domain or IP) and optional port.
- **path** — the resource within that scheme and authority.
- **query** (optional, after `?`) — attribute-value pairs passing extra data.
- **fragment** (optional, after `#`) — a part *within* the resource.

Every URL and every URN is a URI.

## URL — identifier plus location

A **Uniform Resource Locator** is a URI that also specifies *where* the resource is and *how* to retrieve it — a network address. Break down `http://www.mysite.com:80/path/to/mypage.html?product=camera#Section2`:

- **scheme** — `http`; which protocol to use.
- **authority** — `www.mysite.com` (host) and `:80` (port, omitted when it is the scheme's default — 80 for HTTP, 443 for HTTPS).
- **path** — `/path/to/mypage.html`; where the resource sits on the server.
- **query** — `?product=camera`; key-value pairs, joined by `&`, sent to the server.
- **fragment** — `#Section2`; a location within the returned document. Processed by the browser only — **not sent to the server**.

URLs are central to HTTP but also used with FTP, JDBC, and others.

## URN — a persistent name

A **Uniform Resource Name** is a URI that gives a resource a **permanent, location-independent name**. It names the thing uniquely and globally; it does not tell you how to fetch it, and the resource need not still exist. URNs always use the `urn` scheme:

```
urn:isbn:0451450523
```

- **scheme** — always `urn`.
- **NID** (namespace identifier) — `isbn`, `uuid`, etc.: which naming system the rest belongs to.
- **NSS** (namespace-specific string) — the identifier within that namespace.

## The distinction

Your name identifies you uniquely — that is a **URN**. Your street address tells someone how to reach you — that is a **URL**. "Some identifier for you" — name or address — is a **URI**.

A resource can have many URLs (it is reachable from several places) but ideally one URN (one stable name). In everyday web use almost every URI you meet is a URL, which is why the words get used interchangeably — but the split matters wherever an identifier has to outlive the location it currently points at: an ISBN, a DOI, a UUID in a database, a `urn:` in a document format.
