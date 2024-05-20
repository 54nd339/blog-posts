---
title: Nginx and ngrok - Traffic Management and Localhost Tunneling
description: "Nginx is the workhorse that fronts production web traffic - static server, reverse proxy, load balancer, cache - and its event-driven architecture is why. ngrok solves the opposite problem: getting a public URL to a service on your laptop. Plus where HAProxy, Envoy, and Cloudflare Tunnel fit."
date: 2024-05-20
draft: false
slug: /tech/ngnix
tags:
  - Tools
  - Networking
  - Web Infrastructure
---

Two tools that both stand between the internet and a server, at opposite ends of a project's life. **Nginx** fronts production traffic — it's the thing clients actually connect to before your application ever sees a request. **ngrok** does the reverse: it gives the outside world a way to reach a service running on your laptop, for a demo or a webhook test.

This post covers what Nginx does and why its architecture makes it good at it, how ngrok's tunnel works, and the neighbouring tools — HAProxy, Envoy, Cloudflare Tunnel — you'll run into.

## Nginx

Nginx (say "engine-ex") is open-source software that started as a web server and grew into a general traffic handler. Four roles:

**Web server.** It serves static files — HTML, CSS, JS, images, video — extremely efficiently, handling a large number of concurrent connections on very little memory.

**Reverse proxy.** It sits in front of your application servers (Node, Django, Spring, whatever), takes client requests, forwards them to a backend, and returns the response. This hides the backends' addresses and details behind one public face, and it's the natural place to do **TLS termination** — decrypt incoming [HTTPS](/citadel/interview/https) once at the edge and pass plain HTTP to the internal services, so they don't each carry the crypto cost. See [forward and reverse proxies](/citadel/interview/proxies).

**Load balancer.** With several backend instances, Nginx spreads requests across them — round-robin, least-connections, or IP-hash — and health-checks them, pulling unresponsive ones out of rotation. [More on load balancing](/citadel/interview/load-balancing).

**HTTP cache.** It caches backend responses and serves repeat requests for the same content itself, cutting latency and backend load.

### Why it's good at this

Nginx is **event-driven, asynchronous, and non-blocking**. A master process manages a small number of single-threaded worker processes, and each worker handles thousands of connections at once because it never blocks waiting on I/O — it uses the OS's event notification (`epoll` on Linux, `kqueue` on BSD) to be told when a connection is ready to read or write. Contrast the older model of one thread or process per connection, which runs out of memory long before it runs out of connections.

It's configured with a text file, and its stability and low footprint have made it a standard front end for high-traffic sites. [Uber built its Uber Gateway](/citadel/system-design/uber), a dynamic configuration layer, on top of Nginx.

## ngrok

ngrok creates a secure tunnel from a public URL to a service on your local machine, without touching DNS, port forwarding, or firewall rules. It exists to solve problems that need your local server to be publicly reachable:

- **Testing webhooks** — [webhook](/citadel/interview/webhook) senders like Stripe or GitHub need a public URL to POST to; your laptop doesn't have one.
- **Demos** — show a work-in-progress to someone without deploying it.
- **Mobile backend debugging** — give an app on a phone a public endpoint for the API you're building locally.

How the tunnel works:

1. Your app runs on, say, `localhost:3000`.
2. You run `ngrok http 3000`. The ngrok client opens a persistent **outbound** connection to ngrok's cloud service.
3. The cloud service gives you a public URL like `https://<random>.ngrok.io`.
4. A request to that URL hits the ngrok cloud, which forwards it down the tunnel to your client, which forwards it to `localhost:3000`. Responses travel back the same way.

Because the connection is outbound, no router or firewall configuration is needed. ngrok also provides HTTPS on the public side even when your local server is plain HTTP (it terminates TLS), and a local inspection UI at `localhost:4040` showing every request and response through the tunnel — which is the real reason it's a good debugging tool. Paid plans add stable reserved domains and TCP tunnels for non-HTTP traffic. It is a development tool, not a way to host anything.

## The neighbours

**Other reverse proxies and load balancers:** **HAProxy**, a very reliable open-source TCP/HTTP load balancer; **Apache** with `mod_proxy`; **Envoy**, a modern edge and service proxy that's the data plane of service meshes like Istio and Linkerd and is built for managing [inter-service traffic](/citadel/system-design/slack); **Traefik** and **Caddy**, which lead with ease of use and automatic Let's Encrypt certificates and integrate tightly with [Kubernetes](/citadel/tech/k8s) as ingress controllers.

**API gateways** are usually built on reverse-proxy engines (Nginx or Envoy) and add API-level features on top — routing to different services, composition, OAuth2/JWT validation, rate limiting. The distinction from a plain proxy is covered in [API styles and the gateway](/citadel/tech/apis).

**Tunnelling alternatives:** [SSH](/citadel/interview/ssh) port forwarding is built into every SSH client; **Cloudflare Tunnel** makes outbound-only connections from your infrastructure to Cloudflare's edge, exposing services without opening any inbound port.

## Choosing

Nginx (or HAProxy, Caddy, Traefik) for production web serving, proxying, load balancing, and TLS termination at scale — the choice among them comes down to config style and specific features. ngrok for temporary public exposure of something local. An API gateway when you need API management beyond proxying. Envoy when you're running a service mesh.

## The takeaway

Nginx handles the boring, load-bearing job of being the front door to production traffic, and its event-driven architecture is the reason one modest box can hold tens of thousands of connections. ngrok is the development-time inverse — a public URL to your laptop through an outbound tunnel — and its request inspector is worth as much as the tunnel itself.
