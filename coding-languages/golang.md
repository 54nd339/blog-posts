---
title: Go for Web APIs - net/http, Gorilla Mux, and Middleware
description: Go's standard net/http is enough for a small server, but real APIs want path variables, method-based routing, and middleware. Gorilla Mux adds those while staying close to the standard library. Go's strengths for backend work, what net/http lacks, a working Mux server, and how it compares to Chi and Gin.
date: 2020-11-15
draft: false
slug: /coding-languages/golang
tags:
  - Programming Languages
  - Go
  - Web
---

Go is a common choice for backend services, and building an HTTP API is where it is used most. The standard library's `net/http` gets you a working server quickly, but as the API grows you start wanting things `net/http` does not give you cleanly: variables in the URL path, different handlers for `GET` and `POST` on the same path, middleware applied to a group of routes. **Gorilla Mux** adds those without pulling in a full framework. This post covers why Go suits this work, what `net/http` is missing, and how Mux fills the gap.

## Why Go for backend services

- **Performance.** Go is a [compiled language](/citadel/coding-languages/coding-langs) that produces native machine code — fast execution, important under concurrent load.
- **Concurrency built in.** **Goroutines** (lightweight functions scheduled by the Go runtime) and **channels** (typed communication between them) make it straightforward to handle thousands of simultaneous connections without a thread pool.
- **A capable standard library.** `net/http` provides solid HTTP client and server primitives.
- **Single-binary deployment.** Go compiles to one self-contained static executable. Deployment is copy the binary and run it — ideal for containers.
- **Good fit for microservices.** Small footprint, fast startup, and good [gRPC](/citadel/interview/grpc) support.

## What `net/http` routing lacks

The standard router, `http.ServeMux`, matches on path prefixes and little else. Growing APIs tend to need:

- variables in the path (`/users/{id}`);
- routing by HTTP method on the same path (`GET /items` lists, `POST /items` creates);
- matching on host, subdomain, scheme, headers, or query parameters;
- route groups with shared middleware;
- URL generation from named routes.

You can build all of this on `net/http` by hand, but it gets tedious. That is the niche Mux fills.

## Gorilla Mux

**Gorilla Mux** (`github.com/gorilla/mux`) is a widely used HTTP router, part of the Gorilla toolkit. It implements the standard `http.Handler` interface, so it drops into the built-in server and works with any `net/http`-compatible middleware.

- **Path variables** — `/articles/{category}/{id:[0-9]+}`, with optional regex validation, read back via `mux.Vars(r)`.
- **Method matching** — `.Methods("GET")`, `.Methods("POST")` on the same path.
- **Host, scheme, header, and query matching** — route on almost any request attribute.
- **Subrouters** — a router for a path prefix (`/api/v1`), with routes and middleware attached under it.
- **Named routes** — assign a name, generate URLs from it later.
- **Better defaults** for `404 Not Found` and `405 Method Not Allowed`.
- **Middleware** — functions that run in the request/response cycle, before or after the handler, for logging, authentication ([JWT](/citadel/interview/jwt) checks), CORS, compression, panic recovery. Attach them to the whole router, a subrouter, or one route by wrapping `http.Handler`.

## A working server

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"time"

	"github.com/gorilla/mux"
)

func homeHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "Welcome to the Go web service")
}

func productsHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "listing all products...")
}

// Reads path variables from the request.
func productDetailHandler(w http.ResponseWriter, r *http.Request) {
	vars := mux.Vars(r)
	fmt.Fprintf(w, "product detail — category: %s, id: %s\n", vars["category"], vars["id"])
}

func createProductHandler(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusCreated) // 201
	fmt.Fprintln(w, "product created")
}

// Middleware: log each request and how long it took.
func loggingMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		start := time.Now()
		log.Printf("started %s %s from %s", r.Method, r.URL.Path, r.RemoteAddr)
		next.ServeHTTP(w, r)
		log.Printf("finished %s %s in %v", r.Method, r.URL.Path, time.Since(start))
	})
}

func main() {
	r := mux.NewRouter()
	r.HandleFunc("/", homeHandler).Methods("GET")

	// Subrouter for everything under /products.
	products := r.PathPrefix("/products").Subrouter()
	products.HandleFunc("", productsHandler).Methods("GET")             // GET  /products
	products.HandleFunc("", createProductHandler).Methods("POST")      // POST /products
	products.HandleFunc("/{category}/{id:[0-9]+}", productDetailHandler).Methods("GET")

	r.Use(loggingMiddleware) // applies to every route on r

	log.Println("listening on :8080")
	if err := http.ListenAndServe(":8080", r); err != nil {
		log.Fatal(err)
	}
}
```

Run it with `go get github.com/gorilla/mux` then `go run main.go`.

## Around the router

A production API in Go usually also involves:

- **Package structure** — separate packages for handlers, models and database logic, and route definitions.
- **Requests and responses** — reading headers, query parameters, and JSON bodies off `*http.Request`; writing status codes and JSON via `http.ResponseWriter`.
- **Database access** — a driver plus, often, a query builder or a light ORM, for PostgreSQL, MySQL, MongoDB, or Redis.
- **Server-side templates** — `html/template` for HTML responses when the API is not purely JSON.
- **Testing** — the `testing` package with `net/http/httptest` to exercise handlers and middleware directly.
- **Deployment** — the single static binary in a container or on a host.

## Mux versus the alternatives

| Option | What it is | When |
| --- | --- | --- |
| `net/http.ServeMux` | standard-library router | tiny services, few routes |
| Gorilla Mux | powerful router, close to `net/http` | you want path variables and middleware, not a framework |
| Chi | lightweight, composable, idiomatic router | similar to Mux, strong middleware story |
| Gin | full framework: routing, middleware, rendering | you want batteries included and high throughput |
| Echo, Fiber | other full frameworks | preference and benchmarks |

## The one idea to keep

`net/http` is enough to serve HTTP; it is not enough to *organise* a growing API. Gorilla Mux adds path variables, method routing, subrouters, and middleware while still being a plain `http.Handler`, so you get the routing power without committing to a framework's conventions.
