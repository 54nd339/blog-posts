---
title: gRPC - Protocol Buffers and HTTP/2 for Service Calls
description: A framework that makes a call to a remote service look like a local method call, backed by a Protobuf contract and an HTTP/2 transport. The code-generation flow, the four streaming modes, and why it struggles in the browser.
date: 2024-10-18
draft: false
slug: /interview/grpc
tags:
  - System Design
  - Interview Prep
  - APIs
---

Calling another service usually means building an HTTP request, serialising JSON, sending it, parsing the response, and handling the failure modes by hand. **gRPC** — Google Remote Procedure Call — hides all of that: you define the service once, and a generated client stub turns `stub.SayHello(request)` into a network round trip that feels like a local function call.

This covers the define-generate-call flow, the three technologies that make it fast, its streaming modes, and where it does not fit.

![A comparison table of RPC versus RESTful. RPC: strong coupling, binary formats (Thrift, Protobuf, Avro), TCP, high performance, IDL-driven, auto-generated stubs, not human-readable. REST: weak coupling, text formats (XML, JSON), HTTP/1.1 or HTTP/2, lower performance, human-readable and easy to debug.](../images/rpc-v-rest.png "RPC trades REST's loose coupling and readable payloads for a binary format and higher throughput. gRPC is the modern RPC framework in that left column. Source: ByteByteGo.")

## Define, generate, call

**1. Define the contract** in a `.proto` file using **Protocol Buffers** — Google's language-neutral schema for structured data. You declare the service's methods, their request messages, and their response messages.

```protobuf
syntax = "proto3";

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply);
}

message HelloRequest { string name = 1; }
message HelloReply   { string message = 1; }
```

**2. Generate code.** The Protobuf compiler `protoc`, with a gRPC plugin for your language, produces **client stubs** (methods your client calls; they serialise the request and deserialise the reply) and **server skeletons** (an interface you implement with the actual logic).

**3. Call.** The client invokes a stub method. Under it: the client library **marshals** the request to compact Protobuf binary, sends it over **HTTP/2**, the server library **unmarshals** it to native objects and calls your implementation, then the response travels back the same way. The developer sees a function call; the serialisation and transport are hidden.

## Why it is fast

- **Protocol Buffers.** A strongly typed schema compiled to a compact binary encoding — smaller on the wire and quicker to parse than JSON or XML. Benchmarks commonly put Protobuf-over-gRPC around 5&times; faster than JSON-over-REST for message processing.
- **HTTP/2 transport.** Multiplexing — many concurrent requests on one TCP connection with no head-of-line blocking between them; native bidirectional streaming on a long-lived connection; HPACK header compression; a binary framing layer; and flow control.
- **Streaming, as a first-class feature.** Because HTTP/2 supports it, gRPC exposes four call shapes rather than one.

## The four streaming modes

- **Unary** — one request, one response. The ordinary RPC, equivalent to a REST call.
- **Server streaming** — one request, a stream of responses. A large result set, a feed of notifications.
- **Client streaming** — a stream of requests, one response. A file upload, client-side metrics aggregated server-side.
- **Bidirectional streaming** — both sides stream independently over one connection. Chat, collaborative editing, multiplayer game state.

## Where it does not fit

- **The browser.** Browsers do not expose the low-level HTTP/2 control (trailers especially) that gRPC needs. **gRPC-Web** bridges the gap via a proxy (Envoy) that translates browser-friendly requests into real gRPC — an extra hop and extra setup.
- **Debugging.** A binary payload is not something you can read in the network tab or hand-craft with `curl`; you need tooling that decodes Protobuf.
- **Learning curve.** `.proto` files, the code-generation step, and the HTTP/2 and streaming concepts are more to absorb than posting JSON to a route.
- **Ecosystem.** Growing fast, but gateways, mocking, and testing tools are thinner than REST's in places.

## Use it inside, REST at the edge

gRPC's strengths — low latency, a compiled contract, streaming, code generation across languages — line up almost exactly with **internal service-to-service** communication in a [polyglot](/citadel/interview/cloud-native) backend, which is its dominant use, along with mobile and IoT clients where payload size and parse cost matter. Its weaknesses — browser support, readability — are all at the system's edge. The common shape is gRPC between services and [REST](/citadel/interview/rest-api) or [GraphQL](/citadel/interview/graphql) facing the public, with the [encoding post](/citadel/interview/encoding) covering Protobuf itself in more detail.
