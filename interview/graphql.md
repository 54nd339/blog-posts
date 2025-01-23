---
title: GraphQL - One Endpoint, Client-Specified Queries
description: A typed schema and a single endpoint that let the client ask for exactly the fields it needs, across resources, in one request. What it fixes about REST, how resolvers work, and the caching and query-cost problems it introduces.
date: 2025-01-23
draft: false
slug: /interview/graphql
tags:
  - System Design
  - Interview Prep
  - APIs
---

A mobile screen needs a user's name and their last three post titles. Against a [REST API](/citadel/interview/rest-api) that is typically two requests — `/users/{id}` returning the whole profile, then `/users/{id}/posts?limit=3` — one of them over-fetching, both of them round trips. **GraphQL** collapses that to one request for exactly those fields.

GraphQL is a query language plus a server runtime, built at Facebook in 2012 and open-sourced in 2015. It is not a database — it sits in front of your existing databases, REST APIs, and services. Clients send every operation to a single endpoint (`/graphql`, usually over HTTP POST). This covers what it fixes, how it executes a query, and what it costs.

## What it fixes about REST

- **Over-fetching.** A REST endpoint returns a fixed shape. Need two fields, get twenty. A GraphQL query names the fields it wants — `user { name email }` returns only those.
- **Under-fetching.** A composite view needs several REST endpoints and several round trips. A GraphQL query walks relationships — user to posts to comments — and the server returns the whole tree in one response.

## The building blocks

**Schema (SDL).** A strongly typed description of every type, field, and relationship the API exposes. It is the contract both sides code against, and it is self-documenting.

```graphql
type User {
  id: ID!
  name: String
  email: String
  posts: [Post]
}

type Post {
  id: ID!
  title: String
  author: User
}

type Query {
  user(id: ID!): User
  allPosts: [Post]
}
```

**Operations.** *Query* reads data, shaped to mirror the response the client wants. *Mutation* writes — create, update, delete — and can return the changed data. *Subscription* holds a persistent connection and pushes data when a subscribed event fires.

**Resolvers.** The schema says what exists; a **resolver** function per field says how to fetch it — from a database, a REST call, another service, a constant. When a query arrives, the engine calls the resolver for each requested field and assembles the result.

## The costs

- **The N+1 problem.** A query for 50 posts each with an author naively fires 1 query for the posts and then 50 for the authors. The fix is batching — a DataLoader that collects the 50 author IDs and fetches them in one call — and it is not automatic; resolvers must be written for it.
- **Caching is harder.** REST gets HTTP caching for free because a URL identifies a resource. GraphQL's single POST endpoint does not, so caching moves to the client (normalised by object ID) or to the resolver layer.
- **Query cost.** A client can request a deeply nested, expensive query. The server needs depth limits, complexity scoring, and per-client rate limits to avoid being a self-service denial-of-service surface.
- **File uploads** are not in the spec and need an add-on convention.
- **Setup** is heavier than standing up a REST endpoint.

## Deployment patterns

- **Thin layer.** One GraphQL server in front of existing backends, translating queries into calls to them. The easy on-ramp; the risk is the layer accreting into a monolith as it aggregates more sources.
- **Backend-for-frontend (BFF).** Each client type — iOS, Android, web — gets its own GraphQL server tuned to that client's needs, each talking to downstream [services](/citadel/interview/cloud-native). Tailored APIs and team autonomy, at the price of duplicated fetching logic and more servers to operate.
- **Federation.** Each service owns a GraphQL subgraph and can extend types from others; a gateway composes them into one graph (Apollo Federation, or manual schema stitching). Decentralised ownership with a unified graph, at the cost of gateway complexity and cross-team coordination.

## It moves work from the server to the schema

GraphQL's trade is clean to state: the client stops depending on endpoint shapes and starts depending on a typed graph, which is why frontend teams move faster once the schema is stable. What the server gives up is the simplicity REST bought from HTTP — free caching, a trivial mental model of cost per request, an easily inspected payload. If clients are few and their needs are stable, that trade is usually not worth making; if they are many, diverse, and changing, it is. The [gRPC post](/citadel/interview/grpc) covers the other common REST alternative, aimed at a different problem.
