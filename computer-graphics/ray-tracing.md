---
title: Ray Tracing and Global Illumination
description: Rasterization asks "where does this triangle land on screen"; ray tracing asks "what does light do to reach this pixel". Following rays backward from the camera gives reflections, refraction, and soft shadows almost for free - and path tracing, which samples the rendering equation, gives physically correct global illumination at the cost of noise.
date: 2023-06-08
draft: false
slug: /computer-graphics/ray-tracing
tags:
  - Computer Graphics
  - Rendering
---

The [rasterization pipeline](/citadel/computer-graphics/3d) is fast because it asks a local question: for each triangle, which pixels does it cover, and what colour is each? It never naturally knows about the *rest* of the scene — a mirror, the coloured bounce light from a red wall, the soft edge of a shadow — so those effects are bolted on with tricks (shadow maps, environment maps, screen-space reflections), each with artifacts.

Ray tracing asks the opposite, global question: for this pixel, what path did light take to get here? Answering it means simulating light itself, and reflections, refraction, and shadows fall out of the same loop instead of being special cases.

## Casting rays

Shoot a **primary ray** from the camera through each pixel into the scene. Find the nearest surface it hits (ray–object intersection). The colour of that pixel is whatever's happening at that hit point.

Two core intersection tests:

- **Ray–sphere** — substitute the ray `P(t) = O + tD` into `|P − C|² = r²`, get a quadratic in `t`, take the smaller positive root.
- **Ray–triangle** — the Möller–Trumbore algorithm solves for the barycentric coordinates and `t` in one shot; if the barycentrics are in `[0,1]` and sum ≤ 1, and `t > 0`, it's a hit.

At the hit point you evaluate local shading (see [shading and texturing](/citadel/computer-graphics/shading-texturing)) — but ray tracing's power is that you can now spawn *more* rays.

## Whitted ray tracing

Turner Whitted (1980): at each hit, recursively cast

- a **shadow ray** toward each light — if it hits something before reaching the light, the point is in shadow (a crisp, hard shadow for a point light);
- a **reflection ray** in the mirror direction, for shiny surfaces;
- a **refraction ray** bent by Snell's law, for transparent surfaces, with the **Fresnel** term deciding how much light reflects vs transmits at the angle.

Recurse to a depth limit. This gives perfect mirror reflections, glass, and sharp shadows with a few dozen lines of code — effects that rasterization struggles with. What it *doesn't* give is diffuse interreflection (the red wall tinting a white floor), soft shadows from area lights, glossy (blurry) reflections, or caustics. Those need the full model.

## The rendering equation

Kajiya (1986) wrote down what "global illumination" actually means. The outgoing radiance from a point `x` in direction `ωₒ` is the emitted light plus the integral, over every incoming direction `ωᵢ` on the hemisphere, of incoming radiance times the surface's BRDF times a cosine term:

$$L_o(x, \omega_o) = L_e(x, \omega_o) + \int_{\Omega} f_r(x, \omega_i, \omega_o)\, L_i(x, \omega_i)\, (\omega_i \cdot n)\, d\omega_i$$

It's recursive — `Lᵢ` at `x` is `Lₒ` from whatever surface that incoming direction sees — and there's no closed-form solution for a real scene. You approximate the integral by **sampling**.

## Path tracing

**Monte-Carlo path tracing**: to estimate the integral at a hit point, pick a random incoming direction (weighted by the BRDF), cast a ray that way, and recurse. One random path from the camera to a light source is one sample of the light transport. Average many paths per pixel and you converge on the true value.

- **Diffuse bounces** are just more random directions, so colour bleeding, soft shadows (area lights are sampled by picking a random point on them), and glossy reflections all emerge from the same mechanism — no special cases.
- **Russian roulette** — probabilistically terminate paths after a few bounces (and scale up the survivors) so paths don't run forever, without introducing bias.
- The cost is **noise**: with few samples per pixel the image is grainy, and noise falls only as `1/√N`, so halving it costs 4× the rays.

Variance-reduction techniques:

- **Next-event estimation** — at each bounce, also cast a direct shadow ray to a light and add its contribution, instead of hoping a random bounce lands on the light. Massively reduces noise for small lights.
- **Multiple importance sampling** — combine BRDF sampling and light sampling, weighting each by how well it handles the current situation (BRDF sampling wins for sharp highlights, light sampling for small bright lights).
- **Bidirectional path tracing** — trace paths from the camera *and* from the lights and connect them; catches caustics that camera-only paths almost never find.
- **Photon mapping** — a two-pass method: shoot photons from lights and store where they land, then gather nearby photons during a render pass. Strong for caustics and participating media.

## Acceleration structures

A scene has millions of triangles; testing a ray against all of them per pixel is hopeless. A **bounding volume hierarchy** (BVH) is a tree of nested boxes: test the ray against a box, and only descend into its children if it hits. A well-built BVH (using the **surface area heuristic** to choose splits) turns intersection from O(triangles) into O(log triangles). kd-trees and uniform grids are alternatives; BVH is the standard for dynamic scenes because it's cheap to refit when things move.

## Radiosity

An older global-illumination method, restricted to **diffuse** surfaces: divide surfaces into patches, compute **form factors** (how much each patch sees each other patch), and solve the resulting linear system for the equilibrium radiance of every patch. View-independent (you can walk around the solved scene freely) and it nails soft diffuse interreflection, but it doesn't do specular or glossy at all, and the patch discretisation shows. Mostly historical now; path tracing subsumes it.

## Hardware and the state of play

GPUs added **RT cores** (2018) — fixed-function units that do BVH traversal and triangle intersection in hardware, exposed through DirectX Raytracing and Vulkan Ray Tracing. Real-time renderers now do a *hybrid*: rasterize the primary visibility (fast, exact) and ray-trace the hard parts — reflections, shadows, ambient occlusion, a bounce or two of GI — at low sample counts, then run an **ML denoiser** (trained to reconstruct a clean image from noisy 1–few-sample-per-pixel input) and temporal accumulation across frames. **ReSTIR** (spatiotemporal reservoir resampling) makes many-light direct lighting tractable in real time. Offline film rendering has been essentially all path tracing for over a decade.

## The one idea to keep

Ray tracing follows light backward from the camera, so reflection, refraction, and shadows come from recursively casting more rays rather than from screen-space hacks. Whitted ray tracing does the mirror-and-glass part directly; the full picture — colour bleeding, soft shadows, glossy reflections — is the rendering equation, an integral over all incoming directions that you estimate by Monte-Carlo sampling (path tracing), trading a clean closed form for noise that only a denoiser or many samples removes. A BVH is what makes any of it fast enough to run.
