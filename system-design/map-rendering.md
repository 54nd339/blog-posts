---
title: How Digital Maps Work - Tiles, Routing, and Nearby Search
description: Rendering the world without loading one giant image, turning a road network into a shortest-path problem, and answering "restaurants near me" without measuring the distance to every restaurant. Tiling, graph routing, geohashing, and quadtrees.
date: 2025-01-05
draft: false
slug: /system-design/map-rendering
tags:
  - System Design
  - Interview Prep
  - Geospatial
---

A map app does three separable jobs: draw the world, route you across it, and find things near you. Each has a neat core idea — pre-cut image tiles, a road graph, and a spatial index — and this covers all three.

## Rendering: pre-computed tiles

You can't ship one image of the world. Instead, the map is projected to a huge 2D image and cut into small square **tiles**, typically 256×256 pixels, which rarely change. The client downloads only the tiles for the area and zoom level on screen and stitches them into a mosaic.

Tiles are pre-rendered at many **zoom levels** — Google Maps uses around 21. At level 0 the whole world is one 256×256 tile; level 1 splits that into 4 tiles; each level multiplies the tile count (and effective resolution) by about 4, so each step in shows more detail over less ground. The client never downloads more detail than the current view needs, which matters on mobile data.

Tiles are static files, so they're served from a [CDN](/citadel/interview/cdn) backed by object storage — panning and zooming pull the next tiles from a nearby edge.

## Location service

The app periodically sends your latitude and longitude — every few seconds — to a **location service**. Aggregated across many users, that stream detects new or closed roads from movement patterns, improves map accuracy over time, and is the raw input for live traffic.

## Navigation service

Given origin and destination, the **navigation service** finds a fast route, with help from two sub-services:

- **Geocoding service** — translates between addresses and coordinates, both ways (an address to a lat/long, and reverse).
- **Route planner** — the turn-by-turn brain:
  1. **Graph representation.** The road network becomes a graph. Roads are split into **road segments** carrying speed limits, one-way rules, and junction data; segments are nodes, and an edge joins two segments you can drive directly between.
  2. **Pathfinding.** Route A→B is then a shortest-path problem, solved with **Dijkstra's algorithm** or **A\***.
  3. **Travel time.** Shortest isn't fastest — the planner estimates travel time for several candidate paths using live traffic (from the location-service data) and historical traffic by time and day.
  4. **Ranking.** Rank the candidates by predicted time and apply preferences — avoid tolls, ferries, highways.

## Nearby search

A **location-based service** takes your location and a radius and returns nearby points of interest. With millions of POIs, computing the distance to every one is far too slow. Two spatial indexes solve it.

### Geohashing

A **geohash** encodes a lat/long as a short string by recursively dividing the Earth into quadrants and interleaving bits from the latitude and longitude. A longer string is a smaller, more precise cell.

Nearby locations usually share a geohash **prefix**, so "nearby" becomes a prefix query: `SELECT * FROM points_of_interest WHERE geohash LIKE 'tdr1%'`, plus the neighboring cells. That collapses the search space, and geohashes index cleanly in a normal database.

Its weaknesses: two points either side of a cell boundary can have completely different prefixes (the boundary "jump"), and the grid is uniform, so a dense city cell holds thousands of POIs while an ocean cell holds none.

### Quadtrees

A **quadtree** partitions 2D space by recursively splitting each region into four quadrants until a stop condition. For nearby search it's usually an **in-memory** structure on each LBS server, built at startup: the root is the whole service region, and each node splits until a leaf holds at most, say, 100 businesses — so dense areas subdivide deeply and sparse areas stay coarse.

A search descends to the leaf containing your location. If that leaf has enough businesses, return them; if not, expand into neighboring leaves until you have enough.

The operational catch: rebuilding a quadtree for, say, 200 million businesses takes minutes, and a server can't serve traffic while it builds. So roll out new LBS releases or POI-data updates **incrementally, to a few servers at a time**, or you brown out the cluster.

## The takeaway

Each job replaces a brute-force operation with a precomputed index: tiles instead of rendering on demand, a road graph instead of geometry, and a spatial index instead of distance-to-everything. Uber's [H3 hexagonal grid](/citadel/system-design/uber) is another take on that last one.
