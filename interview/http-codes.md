---
title: HTTP Status Codes - The Five Response Classes
description: Every HTTP response carries a three-digit code whose first digit is its class. What 1xx through 5xx mean, the codes worth knowing in each, and the distinctions that trip people up.
date: 2024-05-28
draft: false
slug: /interview/http-codes
tags:
  - System Design
  - Interview Prep
  - HTTP
---

Every HTTP response leads with a three-digit status code, and its first digit is the whole story in miniature: `2` worked, `3` go elsewhere, `4` your request is wrong, `5` the server broke. The other two digits narrow it down. Here are the five classes (per RFC 9110) and the specific codes worth recognising, including the pairs people routinely mix up.

## 1xx — informational

The server got the request and is still working. Rarely surfaced to application code.

| Code | Meaning |
| --- | --- |
| 100 Continue | headers received; send the request body |
| 101 Switching Protocols | switching as the client asked — the WebSocket upgrade |

## 2xx — success

| Code | Meaning |
| --- | --- |
| 200 OK | success; what the body holds depends on the method (a `GET` returns the resource, a `POST` the result of the action) |
| 201 Created | success, and a new resource now exists — typically after a `POST` |
| 202 Accepted | request accepted, processing not finished — an async job was queued |
| 204 No Content | success, deliberately no body — common for `DELETE` and `PUT` |

## 3xx — redirection

| Code | Meaning |
| --- | --- |
| 301 Moved Permanently | the resource has a new URL for good; clients and search engines should update |
| 302 Found | temporary redirect; keep using the original URL next time |
| 304 Not Modified | your cached copy is still current (answer to `If-None-Match` / `If-Modified-Since`); no body sent |

**301 vs 302:** permanent tells caches and crawlers to replace the old link; temporary tells them to keep asking the old URL.

## 4xx — client error

| Code | Meaning |
| --- | --- |
| 400 Bad Request | malformed — bad syntax, invalid framing |
| 401 Unauthorized | actually *unauthenticated* — you have not proven who you are |
| 403 Forbidden | authenticated, identity known, but not allowed this resource |
| 404 Not Found | no such resource (and the server will not say whether it ever existed) |
| 405 Method Not Allowed | the resource exists but does not accept this verb |
| 408 Request Timeout | the client took too long to send the request |
| 429 Too Many Requests | rate limited — slow down, often with a `Retry-After` header |

**401 vs 403:** 401 means "log in"; 403 means "you are logged in and still cannot have this". Sending 401 when you mean 403 invites clients into a pointless re-authentication loop.

## 5xx — server error

| Code | Meaning |
| --- | --- |
| 500 Internal Server Error | an unhandled exception — the catch-all for server bugs |
| 502 Bad Gateway | a proxy or gateway got a garbage response from upstream |
| 503 Service Unavailable | temporarily down — overloaded or in maintenance |
| 504 Gateway Timeout | a proxy or gateway waited for upstream and gave up |

**502/503/504** all point at infrastructure: 502 and 504 mean a [proxy](/citadel/interview/proxies) could not get a good answer from the service behind it, 503 means the service itself is refusing load right now.

## The codes are a contract, so send the right one

The value of status codes is that a client can act on the class without reading the body — retry a 503, refresh auth on a 401, stop on a 400, follow a 301. That only works if servers are honest with them: a 200 wrapping an error message in JSON, or a 404 for something the caller is simply not allowed to see, breaks every client that trusted the number. The full request/response model these codes sit inside is in [HTTP](/citadel/interview/http), and their use in API design is in [REST APIs](/citadel/interview/rest-api).
