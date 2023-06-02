---
title: Text and Images - Encoding, Fonts, Colour Management, and File Formats
description: How text is encoded and styled, what OCR does, the difference between raster and vector images, how digital cameras and scanners capture light, why colour looks different across devices and how ICC profiles fix it, the common image file formats, and the parts of a graphics system.
date: 2023-06-02
draft: false
slug: /computer-graphics/text-image
tags:
  - Computer Graphics
  - Image Processing
  - Multimedia
---

Text and images are the informational core of most multimedia. Both look simple and both hide a stack of decisions: which number represents this character, which pixels represent this photograph, why the same colour value looks different on two monitors, and which file format to save it in. This post works through those decisions, then closes with the hardware and software that make up a graphics system.

It continues the [multimedia overview](/citadel/computer-graphics/multimedia).

## Text

**Types.** *Plain text* is unformatted characters — a `.txt` file, small and universally readable. *Formatted (rich) text* carries styling: fonts, sizes, colours, bold, alignment. *Hypertext* adds links that let the reader jump non-linearly; the web is built on it.

**Encoding.** Computers store characters as numbers.

- **ASCII** uses 7 or 8 bits — enough for English letters, digits, and basic punctuation, but not accented characters or other scripts.
- **Unicode** aims to cover every writing system plus symbols and emoji. **UTF-8** dominates the web because it is backward-compatible with ASCII and compact for Latin text; **UTF-16** is common in memory. Unicode is why English, Hindi, and Chinese can share a page.

**Fonts.** A **typeface** is the design of the letterforms (Times New Roman, Helvetica). A **font** is a specific instance — a size, weight, and style. Families:

- **Serif** — small strokes at letter ends (Times New Roman, Garamond); traditional for long print passages.
- **Sans-serif** — no serifs (Arial, Verdana); cleaner, common on screens.
- **Script** — imitates handwriting.
- **Display / decorative** — stylised, for headlines and short attention-grabbing text.

**Insertion.** In production, text is typed into fields, imported from files, generated dynamically from a database or user input, or rendered as part of a graphic.

**OCR.** **Optical character recognition** converts a scanned image of text into machine-encoded characters: the software analyses the image, identifies characters by their shapes, and outputs ASCII or Unicode. It powers book digitisation, searchable PDFs, data-entry automation, and text-to-speech for accessibility.

**Text file formats.** `.txt` (plain, smallest, universal), `.rtf` (rich text, widely readable), `.doc`/`.docx` (Microsoft Word, rich formatting and embedded objects), `.pdf` (fixed layout across platforms, ideal for distribution and printing).

## Raster and vector images

- **Bitmap (raster)** images are a grid of pixels, each with a colour value. Photographs, scans, detailed paintings. They are **resolution-dependent** — scaling up past the native resolution produces visible pixelation. Formats: JPEG, GIF, PNG, BMP, TIFF.
- **Vector** images store mathematical paths — lines, curves, shapes — as instructions for drawing, not pixels. Logos, icons, illustrations, technical drawings. They are **resolution-independent**: scale to any size without loss, and often smaller for simple graphics. Formats: SVG, AI, EPS.

A logo that must sit on a transparent background at any size wants a vector format, or PNG if it must be raster.

## Colour

The colour *models* — RGB (additive, for displays), CMYK (subtractive, for print), HSV/HSL (hue, saturation, value — intuitive for pickers), and greyscale — are covered with their conversions in [3D graphics](/citadel/computer-graphics/3d). What matters here is that RGB and CMYK are **device-dependent**: the same numbers produce different colours on different hardware, because each device has its own **gamut** — the range of colours it can reproduce.

**Colour management systems** aim for consistent appearance across devices. An **ICC profile** describes the colour characteristics of one device (a specific camera, monitor, or printer), and the CMS uses the profiles to translate colours from one device's space to another's, preserving the visual intent as far as the gamuts allow.

**Device-independent colour models** define colour by human perception rather than by any device. **CIELAB** ($L^*$ for lightness, $a^*$ for the green–red axis, $b^*$ for the blue–yellow axis) and **CIEXYZ** (based on the eye's response) serve as the reference space that conversions pass through.

**Gamma** is the relationship between a pixel's stored numeric value and its perceived brightness on screen. Human vision and display devices are both non-linear in brightness, so **gamma correction** adjusts the tones to display correctly — neither crushed dark nor washed out.

## Capturing images

- **Scanners.** A light source illuminates the document; a sensor (a CCD — charge-coupled device — or a contact image sensor) moves across it, converting reflected light to electrical signals that are then digitised. Flatbed (document on glass) or sheet-fed (rollers).
- **Digital cameras.** Light passes through the lens onto an image sensor (CCD or CMOS — complementary metal-oxide semiconductor), which converts photons to charge; the camera digitises and processes that into an image file. Quality depends on resolution (megapixels), lens, sensor size, and ISO sensitivity.

**Interface standards** connect these devices: **USB** (the common one — see [I/O organisation](/citadel/computer-architecture/io-organisation)), **Thunderbolt** (high-speed), and **TWAIN**, which is not a physical interface but an API and protocol that lets software like Photoshop control a scanner or camera and pull images from it directly.

## Specifying a digital image

- **Resolution** — pixels per inch (PPI) for displays, dots per inch (DPI) for printers. Higher means finer detail.
- **Pixel dimensions** — total pixels horizontally and vertically, e.g. 1920 × 1080.
- **Bit depth (colour depth)** — bits per pixel: 1-bit (2 colours), 8-bit greyscale (256 shades), 8-bit indexed colour (256 colours), 24-bit true colour (8 bits per RGB channel, ~16.7 million colours), and higher for HDR work.
- **Aspect ratio** — the width-to-height proportion, e.g. 4:3 or 16:9.

## Image processing steps

A typical processing sequence: **acquisition** (camera or scanner), **preprocessing** (noise reduction, geometric correction), **enhancement** (brightness, contrast, sharpening), **segmentation** (separating regions or objects), **analysis / feature extraction** (measuring properties), and **output**. Common software: Adobe Photoshop and GIMP for raster editing, Illustrator and Inkscape for vector, Lightroom for photo management. The algorithms are covered in [digital image processing](/citadel/computer-graphics/image-processing).

## Image file formats

| Format | Compression | Transparency | Best for |
| --- | --- | --- | --- |
| **JPEG** | Lossy | None | Photographs, continuous-tone images |
| **GIF** | Lossless (LZW), ≤256 colours | 1-bit (on/off) | Simple graphics, short animations |
| **PNG** | Lossless | Full alpha channel | Web graphics, logos, images needing partial transparency |
| **TIFF** | Lossless or lossy | Yes | Print and professional photography; layers, high bit depth; large files |
| **BMP** | None or simple RLE | No | Uncompressed Windows bitmaps; large |
| **SVG** | Vector (XML) | Yes | Scalable web graphics, with interactivity and animation |

The compression techniques each format uses — RLE, LZW, DCT, Huffman — are covered in [compression](/citadel/computer-graphics/compression).

**Output.** A monitor displays in RGB (additive), mapping image pixels to screen pixels. A printer works in CMYK (subtractive), so RGB image data is converted to CMYK — and because the CMYK gamut is generally smaller, some colours shift in the process. Print quality depends on the printer's DPI.

## Parts of a graphics system

| Part | Role |
| --- | --- |
| **Input devices** | Mouse, keyboard, stylus, scanner, camera |
| **CPU** | General computation |
| **GPU** | Specialised processor that rapidly manipulates memory to build images in the frame buffer; essential for 3D and video |
| **RAM** | System memory for the CPU |
| **VRAM** | Dedicated GPU memory holding image data, textures, and the frame buffer |
| **Frame buffer** | The region of VRAM holding the pixels currently on screen; the GPU writes it, the display controller reads it to refresh the display |
| **Output devices** | Monitor, projector, printer |
| **Software** | OS, device drivers, graphics APIs (DirectX, OpenGL, Vulkan, Metal), applications |

## The one idea to keep

Text and images are numbers under a chosen convention — a character encoding, a pixel grid, a colour model — and most of the practical difficulty is that the conventions do not agree across devices. Unicode fixed it for text; ICC profiles and device-independent colour spaces are the equivalent fix for colour, translating everything through a common reference so a photograph looks the same on the camera, the monitor, and the print.
