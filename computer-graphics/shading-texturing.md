---
title: Shading Models and Texture Mapping
description: Shading turns a surface point, a light, and a viewer into a colour. The Phong model does it with an ad-hoc diffuse-plus-specular formula; physically based rendering does it with a microfacet BRDF that conserves energy. Texture mapping supplies the per-point detail, and filtering is what keeps it from shimmering.
date: 2023-06-07
draft: false
slug: /computer-graphics/shading-texturing
tags:
  - Computer Graphics
  - Rendering
  - Image Processing
---

Once the [rasterizer or ray tracer](/citadel/computer-graphics/3d) knows a pixel corresponds to a particular point on a particular surface, something has to decide what colour that pixel is. That's **shading**: a function of the surface's material, the surface normal, the directions to the lights, and the direction to the viewer. And because a bare geometric surface has no detail, **texture mapping** supplies the per-point variation — colour, bumpiness, shininess — from images wrapped onto the geometry.

This post is the shading models from the 1970s ad-hoc formula to today's physically based one, and how texturing works without turning into a shimmering mess.

## The local lighting terms

Classic real-time shading splits reflected light into three parts:

- **Ambient** — a constant fudge for "light bouncing around the scene that we're not simulating." A flat fill so shadows aren't pure black.
- **Diffuse (Lambertian)** — light scattered equally in all directions by a matte surface. Its intensity is `max(0, n · l)` — the cosine of the angle between the surface normal `n` and the light direction `l`. Brightest when the light hits head-on, zero at grazing angles. View-independent.
- **Specular** — the shiny highlight, concentrated near the mirror-reflection direction. In the **Phong** model it's `max(0, r · v)^s` where `r` is the reflected light direction, `v` points to the viewer, and the exponent `s` (shininess) tightens the highlight. **Blinn–Phong** replaces `r · v` with `n · h`, where `h` is the halfway vector between `l` and `v` — cheaper and better-behaved at grazing angles, and the default in the fixed-function era.

Total colour = ambient + Σ over lights (diffuse + specular), each scaled by the light's colour and intensity, with distance attenuation.

## Shading frequency

*Where* you evaluate the shading formula matters:

- **Flat shading** — once per triangle, using the face normal. Faceted look.
- **Gouraud shading** — once per vertex, then interpolate the resulting *colours* across the triangle. Cheap, but misses highlights that fall between vertices.
- **Phong shading** — interpolate the *normals* across the triangle and evaluate the formula per pixel. More expensive, correct highlights. This is what every modern fragment shader does (don't confuse Phong *shading*, the per-pixel evaluation, with the Phong *reflection model*, the specular formula).

## Physically based rendering

Phong looks fine but isn't grounded in physics: it can reflect more light than it receives, its parameters don't correspond to real material properties, and it doesn't respond correctly under varied lighting. **PBR** fixes this by building the specular term from a **microfacet BRDF**.

The idea: a rough surface is a sea of tiny perfect mirrors (**microfacets**) at random orientations. The Cook–Torrance BRDF specular term is:

$$f_r = \frac{D \, F \, G}{4 \, (n \cdot l)(n \cdot v)}$$

- **D — normal distribution function** — what fraction of microfacets are oriented to reflect `l` toward `v`. Controlled by **roughness**: smooth = a tight distribution = a sharp highlight; rough = spread out = a broad, dim highlight. GGX/Trowbridge–Reitz is the standard `D`.
- **F — Fresnel** — reflectance rises toward 100% at grazing angles (everything becomes mirror-like at a shallow angle). The Schlick approximation is used everywhere.
- **G — geometry / masking-shadowing** — microfacets block each other at grazing angles; `G` accounts for that so energy is conserved.

The workflow artists use is **metallic/roughness**: a **base colour**, a **metallic** flag (0 = dielectric, 1 = metal — metals have no diffuse term and a coloured specular), and a **roughness** value. These map to real-world material measurements, so an asset looks right under any lighting — which is the whole point.

**Image-based lighting** completes it: instead of a few point lights, light the surface with a captured environment (an HDR panorama), prefiltered per roughness level so a rough surface samples a blurred version. This is why PBR assets sit convincingly in real photographed backgrounds.

## Texture mapping

A **texture** is an image sampled to provide per-point surface data. Each vertex carries **UV coordinates** — a 2D position in `[0,1]²` texture space — and these interpolate across the triangle so every pixel gets a `(u, v)` to look up.

The subtlety in a rasterizer: interpolating `(u, v)` *linearly* in screen space is wrong under perspective — a receding checkerboard floor would look warped. The fix is **perspective-correct interpolation**: interpolate `u/w`, `v/w`, and `1/w` linearly, then divide. Hardware does this automatically.

Texture uses beyond colour:

- **Albedo / diffuse map** — the base colour.
- **Normal map** — encodes a perturbed surface normal per texel (in tangent space), so a flat polygon shows bumps and grooves under lighting without extra geometry. **Bump maps** (a heightfield) are the older, cruder version; **parallax** and **displacement** maps actually shift the apparent or real surface.
- **Roughness / metallic / ambient-occlusion maps** — spatial variation of the PBR parameters.
- **Environment / cube maps** — the surrounding scene, sampled by the reflection vector for reflections and IBL.

## Filtering and aliasing

A screen pixel doesn't map to one texel — it maps to a *region* of the texture, and that region varies wildly across the frame (huge near the camera, sub-texel in the distance). Point-sampling one texel per pixel gives blocky close-ups and a shimmering, crawling mess in the distance as the sample point jitters between texels frame to frame.

- **Bilinear filtering** — blend the 4 nearest texels. Fixes close-up blockiness.
- **Mipmapping** — precompute a pyramid of half-resolution versions of the texture. For a pixel covering many texels, sample the mip level whose texels are about pixel-sized. Kills the distant shimmer; costs 33% more memory.
- **Trilinear** — blend between two mip levels so there's no visible seam where the level switches.
- **Anisotropic filtering** — a pixel footprint on a surface viewed at a steep angle is long and thin, not square; plain mipmapping over-blurs along the short axis. Anisotropic filtering takes several samples along the footprint's long axis, keeping oblique surfaces (roads, floors) sharp. The main "AF 16x" quality setting in games.

## Other maps and effects

- **Shadow maps** — render the scene depth from the light's view; a point is in shadow if its distance from the light exceeds the stored depth. Cheap, ubiquitous, prone to acne and peter-panning without bias tuning; PCF and variance shadow maps soften the edges.
- **Environment maps** for cheap reflections on curved surfaces.
- **Lightmaps** — precomputed global illumination baked into a texture for static geometry.

## The one idea to keep

Shading is a function from (material, normal, light, view) to colour: Blinn–Phong does it with an ad-hoc diffuse cosine plus a halfway-vector specular, evaluated per pixel with interpolated normals. Physically based rendering replaces the specular guess with a microfacet BRDF (the `D·F·G` terms) parameterised by roughness and metalness, so materials stay correct under any lighting. Textures supply the per-point detail via perspective-correct UV interpolation, and mipmapping plus anisotropic filtering are what stop that detail from shimmering in the distance.
