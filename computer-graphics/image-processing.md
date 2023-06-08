---
title: Digital Image Processing - Enhancement, Restoration, and Recognition
description: What a digital image is as a matrix of numbers, the enhancement transforms that make it look better, the restoration model that reverses known degradation, morphological operations on shape, segmentation methods that partition an image, and the pipeline from pixels to object recognition.
date: 2023-06-08
draft: false
slug: /computer-graphics/image-processing
tags:
  - Computer Graphics
  - Image Processing
---

A digital image is a matrix of numbers, and **digital image processing** is the arithmetic you do on that matrix — to make it clearer, to reverse a known distortion, to isolate the objects in it, and ultimately to recognise what they are. This post works through the stages: fundamentals, enhancement, restoration, morphology, segmentation, compression, and recognition.

It goes deeper than the processing overview in [text and image](/citadel/computer-graphics/text-image).

## Fundamentals

Image processing shows up in medical imaging, remote sensing, machine vision, forensics, art and entertainment, and document processing (OCR). In each, a **digitised image** is a numerical representation of a 2D scene, produced by two steps:

- **Sampling** — converting continuous spatial coordinates into a discrete grid of pixels.
- **Quantisation** — converting each sample's continuous brightness into a discrete level.

Sampling decides *where* you measure; quantisation decides *how precisely*.

**Image types:** binary (each pixel 0 or 1), greyscale (an intensity, typically 0–255), colour (multiple values per pixel — RGB, HSV). **Spatial resolution** is the pixel count; **intensity resolution** (colour depth) is the bits per pixel — 8-bit greyscale gives $2^8 = 256$ shades. Pixels relate through **neighbours** (4-connected, 8-connected, diagonal) and, from those, **adjacency, connectivity, paths, regions, and boundaries** — the vocabulary for grouping pixels into objects.

The mathematical tools are [linear algebra](/citadel/maths/linear-algebra) (images as matrices), the **Fourier transform** (moving between the spatial and frequency domains), **convolution** (applying a filter kernel), and probability and statistics (noise models, segmentation).

## Enhancement

Enhancement is **subjective** — making an image more suitable for a purpose or more pleasing to a viewer.

**Intensity transformations** map each pixel value $r$ to a new value $s$:

- **Negative** — $s = L - 1 - r$; inverts intensities, useful for seeing detail in dark regions of medical images.
- **Log** — $s = c\log(1 + r)$; compresses the dynamic range of images with a wide intensity spread.
- **Power-law (gamma)** — $s = c\,r^{\gamma}$; used for display gamma correction and contrast adjustment.
- **Contrast stretching** — expands the used range of intensities to span the full scale.

The log and gamma curves are why the [logarithm](/citadel/maths/logarithms) appears in tone mapping.

**Histogram processing** works on the distribution of intensities. **Histogram equalisation** spreads out the most frequent values, usually improving global contrast; **histogram matching** reshapes an image's histogram to a target.

**Spatial filters** operate on pixel neighbourhoods. **Smoothing** filters (averaging, Gaussian) blur and reduce noise. **Sharpening** filters highlight edges, using first derivatives (the gradient — Sobel, Prewitt) or the second derivative (the Laplacian); **unsharp masking** and highboost filtering subtract a blurred copy from the original to sharpen.

**Frequency-domain filters** modify the Fourier transform: lowpass filters (Ideal, Butterworth, Gaussian) attenuate high frequencies to smooth; highpass filters attenuate low frequencies to sharpen.

## Restoration

Restoration is **objective** — recovering an image degraded by a process you can model. The standard degradation model is

$$g(x, y) = h(x, y) * f(x, y) + \eta(x, y)$$

where $f$ is the original image, $h$ is the degradation (a blur), $\eta$ is additive noise, $*$ is convolution, and $g$ is what you observe. The goal is to estimate $f$.

**Noise models:** Gaussian (from sensor electronics), salt-and-pepper (impulse noise — random black and white pixels), plus Rayleigh, gamma, exponential, and uniform. Choosing the right filter depends on knowing the noise type.

- **Mean filters** — arithmetic, geometric, harmonic, contraharmonic; each handles different noise.
- **Order-statistics filters** — based on ranking the values in a neighbourhood. The **median filter** removes salt-and-pepper noise while preserving edges better than a mean filter; max, min, and midpoint filters exist too.
- **Adaptive filters** — behaviour changes with local image statistics; the adaptive median filter grows its window where needed.
- **Notch filters** — periodic noise (from electrical interference) shows up as spikes in the frequency domain, and a notch filter removes those specific frequencies.

## Morphological processing

Morphology deals with the shape and geometric structure of objects, using a small template called a **structuring element** to probe the image. It is defined for binary images and extends to greyscale. The two primitives:

- **Erosion** ($A \ominus B$) — shrinks objects, stripping pixels from their boundaries; removes small noise specks.
- **Dilation** ($A \oplus B$) — grows objects, adding boundary pixels; fills small gaps.

Compounds:

- **Opening** ($A \circ B = (A \ominus B) \oplus B$) — erosion then dilation; smooths contours, breaks thin bridges, removes small protrusions and isolated noise.
- **Closing** ($A \bullet B = (A \oplus B) \ominus B$) — dilation then erosion; fills small holes and fuses narrow breaks.

Other operations: the **hit-or-miss transform** detects a specific configuration of foreground and background pixels; **boundary extraction** is $\beta(A) = A - (A \ominus B)$; **pruning** removes spurious branches left by skeletonisation; and an opening followed by a closing gives **morphological smoothing**.

## Segmentation

Segmentation partitions an image into meaningful regions — the first step toward understanding it.

**Detecting discontinuities:**

- **Point detection** — isolate pixels much brighter or darker than their neighbours, using a mask.
- **Line detection** — masks tuned to horizontal, vertical, or diagonal lines.
- **Edge detection** — locate boundaries between regions. First-order methods (Sobel, Prewitt, Roberts) approximate the gradient; second-order methods use the Laplacian, with **Marr-Hildreth** (Laplacian of Gaussian) smoothing first to control noise. The **Canny detector** is a multi-stage pipeline: Gaussian smoothing, gradient computation, non-maximum suppression to thin edges, and hysteresis thresholding (two thresholds) to link them.

**Thresholding.** Pick a value $T$; pixels with intensity above $T$ are foreground, the rest background. **Global** thresholding uses one $T$ for the whole image; **Otsu's method** chooses $T$ automatically by maximising the between-class variance; **adaptive** thresholding lets $T$ vary across the image for uneven lighting; **multivariable** thresholding uses several channels.

**Region-based.** **Region growing** starts from seed pixels and adds neighbours that meet a similarity criterion. **Region splitting and merging** recursively splits the image (often as a [quadtree](/citadel/data-structures/data-structures)) until each sub-region is homogeneous, then merges adjacent similar regions.

## Compression, briefly

The compression techniques — Huffman, arithmetic, RLE, LZW, and transform coding with DCT and quantisation — are covered in [compression](/citadel/computer-graphics/compression). A few points specific to images:

- **Entropy** from [information theory](/citadel/miscs/information-theory) sets the theoretical lower bound on bits per symbol.
- **Fidelity criteria** measure quality loss: objective measures like root-mean-square error and peak signal-to-noise ratio (PSNR), and subjective opinion scores.
- **Bit-plane coding** decomposes a greyscale image into binary images — one per bit of depth — and compresses each, often with run-length coding.
- **Digital watermarking** embeds imperceptible information for copyright, authentication, or tamper detection — a form of steganography.

## Object recognition

Recognition identifies and locates objects in an image. A typical system is a pipeline: acquisition, preprocessing, segmentation to isolate candidates, **feature extraction**, **classification**, and post-processing.

A **pattern** is a set of features describing an object; a **pattern class** is a group of patterns sharing properties (all cats, all cars). Objects are represented as feature vectors, symbol strings, or graphs. **Feature selection** picks measurable properties that separate classes — colour, texture, shape descriptors (area, perimeter, moments), and detectors like SIFT, SURF, and HOG. Image processing is a prerequisite: enhancement, segmentation, and feature extraction prepare the image and pull out the information the classifier needs.

Two approaches:

- **Statistical (decision-theoretic)** — objects are feature vectors, and a classifier trained on labelled examples learns decision boundaries. **Bayes parametric classification** assumes each class's feature vectors follow a known distribution and assigns an object to the class with the maximum posterior probability, $P(C_i \mid \mathbf{x}) = \dfrac{P(\mathbf{x} \mid C_i)\,P(C_i)}{P(\mathbf{x})}$. See [machine learning](/citadel/artificial-intelligence/ml).
- **Syntactic (structural)** — objects are compositions of simpler primitives and their relations, described by a grammar. **Shape numbers** encode a boundary as a sequence of direction codes; **string matching** compares those descriptions.

**Face recognition** is the canonical hard case. Early methods like **Eigenfaces** project faces onto a lower-dimensional space with [PCA](/citadel/artificial-intelligence/dimesionality-reduction); modern systems use [convolutional neural networks](/citadel/artificial-intelligence/deep-networks).

## The one idea to keep

Every stage of image processing is a neighbourhood operation on a matrix of numbers. Enhancement remaps intensities or filters neighbourhoods to look better; restoration inverts a known degradation model; morphology probes shape with a structuring element; segmentation groups pixels by discontinuity or similarity; and recognition turns the segmented regions into feature vectors a classifier can label. The maths is convolution, thresholds, and set operations, applied over and over across the grid.
