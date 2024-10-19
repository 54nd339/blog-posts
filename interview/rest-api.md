---
title: REST APIs - The Six Constraints and How to Design One
description: REST is an architectural style, not a protocol — resources at URLs, manipulated with HTTP methods, under six constraints. What each constraint buys, the verb semantics, and the design practices that make an API usable and safe.
date: 2024-10-19
draft: false
slug: /interview/rest-api
tags:
  - System Design
  - Interview Prep
  - APIs
---

**REST** — Representational State Transfer — is not a technology or a strict protocol like SOAP. It is an architectural style: a set of constraints for designing networked applications around **resources**. A resource is any addressable thing — a user, a product, an order — identified by a URL, and the client works with **representations** of it, most often JSON. An API that follows the constraints is "RESTful".

This covers the six constraints and what each is for, the HTTP verb semantics, and the practices that separate a good REST API from a Wikipedia-summary one.

## The six constraints

1. **Client-server** — separate the UI/UX concern (client) from data and processing (server) so each evolves independently.
2. **Statelessness** — every request carries everything needed to process it; the server keeps no session state between requests. Any server instance can then handle any request, which is what makes horizontal scaling straightforward.
3. **Cacheability** — responses declare whether they can be cached, so clients and intermediaries ([CDNs](/citadel/interview/cdn), proxies) can serve repeats without hitting the origin.
4. **Layered system** — the client talks to whatever layer is in front of it — a [load balancer](/citadel/interview/load-balancing), a gateway, a [proxy](/citadel/interview/proxies) — without knowing whether it is the end server.
5. **Uniform interface** — the defining constraint, in four parts: resources identified by URIs (`/users/123`); resources manipulated through representations (send back a modified JSON object to change it); self-descriptive messages (a `Content-Type` tells the receiver how to read the body); and **HATEOAS** — hypermedia as the engine of application state — responses include links to the actions available next.
6. **Code on demand** (optional) — the server may ship executable code (JavaScript) to extend the client. The only optional constraint.

## The anatomy of a call

- **Endpoint URL** — identifies the resource: `https://api.example.com/v1/users`.
- **Method** — the action (below).
- **Headers** — `Content-Type`, `Authorization`, and the rest.
- **Request body** — data from client to server, on `POST`/`PUT`/`PATCH`.
- **Response body** — the resource representation, with a [status code](/citadel/interview/http-codes).

## The verbs and their semantics

| Method | Does | Idempotent |
| --- | --- | --- |
| GET | retrieve a representation | yes |
| POST | create a new resource | no — repeats create duplicates |
| PUT | replace a resource (or create at a known URI) | yes |
| PATCH | partially modify a resource | no in general |
| DELETE | remove a resource | yes |
| HEAD | GET without the body | yes |
| OPTIONS | list the methods a resource allows | yes |

Idempotency is the practical property: a client can safely retry a failed `GET`, `PUT`, or `DELETE`; a retried `POST` needs an idempotency key to avoid creating the thing twice.

## Design practices

- **Nouns, plural, for URIs** — `/users`, `/orders`; never `/getUsers`. The verb is the HTTP method.
- **Version in the path** — `/v1/users`, so a breaking change ships as `/v2` without breaking existing clients.
- **Honest status codes** — `201` for a create, `404` for missing, `409` for a conflict, `429` for rate-limited. Do not return `200` with an error inside.
- **HTTPS always**, **authentication and authorization** (OAuth 2.0 is the common standard — see [identity management](/citadel/interview/identity-management)), and **input validation** on everything.
- **Rate limiting** to protect against abuse and runaway clients.
- **Pagination, filtering, sorting** for any endpoint that returns a collection.
- **Idempotency keys** on `POST`s that might be retried.
- **Documentation** — an OpenAPI/Swagger spec.

## The constraints are what make it interoperable

REST's payoff is that any HTTP client in any language already knows how to talk to it — the verbs, the status codes, the caching semantics are all standard, so there is nothing bespoke to learn per API. That only holds while you respect the constraints: statelessness so a load balancer can spread requests, honest status codes so clients can branch on the class, nouns and standard verbs so the shape is predictable. Break them and you have RPC over HTTP wearing REST's clothes. [GraphQL](/citadel/interview/graphql) and [gRPC](/citadel/interview/grpc) are the two common alternatives, each trading some of that universality for a different benefit.
