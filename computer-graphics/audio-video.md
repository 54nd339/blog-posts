---
title: Audio and Video - From Sound Waves to Digital Codecs
description: The physics of sound and how it is digitised with sampling rate and bit depth, what MIDI is and is not, the audio and video processing pipelines, analog video signal formats and TV standards, and the digital video codecs and container formats in use today.
date: 2023-06-03
draft: false
slug: /computer-graphics/audio-video
tags:
  - Computer Graphics
  - Multimedia
---

Audio and video are the media that carry time. A still image is a snapshot; sound and moving pictures only exist as they unfold, which is why both need a sampling rate, a frame rate, and careful synchronisation. This post covers the physics of sound, how audio and video become digital, the processing pipelines for each, and the codecs and file formats in use today.

It continues the [multimedia overview](/citadel/computer-graphics/multimedia).

## The physics of sound

Sound is a **wave** — a vibration travelling through a medium such as air. A vibrating object pushes on nearby air molecules, and the disturbance propagates outward as a longitudinal wave of alternating **compression** (high pressure) and **rarefaction** (low pressure). Its properties:

- **Amplitude** — the size of the pressure swing, perceived as loudness.
- **Frequency** — vibrations per second, in hertz (Hz), perceived as **pitch**. Human hearing spans roughly 20 Hz to 20 kHz.
- **Wavelength** — the distance between consecutive compressions.
- **Timbre** — the quality that distinguishes a trumpet from a violin on the same note, set by the mix of the fundamental frequency and its **overtones** (harmonics).

A **tone** is a pure single frequency (a sine wave); most real sounds are combinations of tones. A **note** in music is a specific pitch and duration. **Musical sound** has regular, periodic vibration and a discernible pitch; **noise** has irregular, random vibration and none. See [waves](/citadel/physics/waves) for the underlying physics.

**Psychoacoustics** studies how people actually perceive sound, which is not a direct readout of the physics — sensitivity varies with frequency, peaking around the range of speech. The **decibel** (dB) is a logarithmic unit for sound level, logarithmic because hearing is: a 10 dB increase is perceived as roughly a doubling of loudness. 0 dB is the threshold of hearing, a whisper around 30 dB, conversation around 60 dB, a rock concert 110–120 dB. The [logarithm](/citadel/maths/logarithms) is why the scale compresses such a wide range.

## From sound to hardware

- **Microphone** — a transducer converting sound to an electrical signal. **Dynamic** mics use electromagnetic induction (a diaphragm-mounted coil moves in a magnetic field); rugged, good for loud sources. **Condenser** mics use a capacitor whose capacitance changes as the diaphragm vibrates; more sensitive and detailed, usually needing phantom power. **Polar patterns** (omnidirectional, cardioid, bidirectional) describe directional sensitivity.
- **Amplifier** — boosts the weak microphone signal to a level that can drive speakers or be recorded cleanly.
- **Speaker** — a transducer doing the reverse of a microphone: an audio signal drives a voice coil in a magnetic field, vibrating a cone to produce sound waves.

## Digital audio

Analog sound must be converted to numbers by **analog-to-digital conversion**. Three specifications set the quality:

- **Sampling rate** — measurements per second. CD quality is 44.1 kHz; professional work uses 48, 96, or higher. A higher rate captures higher frequencies faithfully.
- **Bit depth** — bits per sample, setting the dynamic range. CD is 16-bit (65,536 amplitude levels); professional is 24 or 32-bit.
- **Channels** — mono (1), stereo (2), or surround (5.1, 7.1).

These are the audio form of the sampling and quantisation covered in the [multimedia overview](/citadel/computer-graphics/multimedia).

**Synthesizers** generate audio directly rather than recording it, through methods such as subtractive, additive, FM, and wavetable synthesis.

**MIDI** (Musical Instrument Digital Interface) is *not* audio. It is a protocol carrying performance data — note pitch, velocity, and duration, control signals, and timing — between instruments and computers. MIDI files are tiny (instructions, not waveforms) and easy to edit, but you need a synthesizer or sound card to turn the messages into sound.

A **sound card** (audio interface) handles analog-to-digital conversion for recording, digital-to-analog for playback, often a MIDI interface, connections for microphones and speakers, and sometimes onboard effects.

## Audio processing

A raw recording is usually processed before release:

1. **Editing** — cutting, trimming, arranging clips.
2. **Mixing** — combining tracks, setting levels, panning.
3. **Normalisation** — bringing overall volume to a standard level.
4. **Equalisation** — adjusting the balance of frequency bands.
5. **Dynamics** — **compression** (reducing the loud-to-quiet range), **limiting** (a hard ceiling), **gating** (cutting sound below a threshold, to remove background noise).
6. **Effects** — reverb, delay, chorus, distortion.
7. **Noise reduction** — removing hiss and hum.
8. **Mastering** — final polish for consistent loudness and tone across playback systems.

This is done in a digital audio workstation: Audacity, Adobe Audition, Pro Tools, Logic Pro, Ableton Live, FL Studio.

**Audio file formats:**

| Category | Formats | Trait |
| --- | --- | --- |
| Uncompressed | `.wav` (PCM), `.aiff` | Full quality, large |
| Lossless compressed | `.flac`, `.alac` | Smaller, bit-exact on decode |
| Lossy compressed | `.mp3`, `.aac`, `.ogg` (Vorbis), `.wma` | Much smaller, some quality discarded |

## Video: frames and frame rate

A video is a sequence of still images (**frames**) with synchronised audio. **Frame rate** is frames displayed per second (fps): cinema 24, PAL television 25, NTSC 29.97 (often called 30), and modern digital video 30, 50, 60, or higher. Too low a frame rate makes motion look juddery.

## Analog video

An **analog video camera** focuses light onto an image sensor (a CCD or a vidicon tube), which converts the light pattern for each scan line into an electrical signal recorded to magnetic tape.

**Analog signal formats** differ in how they carry brightness (luminance) and colour (chrominance):

- **Composite** — everything in one signal; simple, but prone to colour bleeding. RCA (yellow) connector.
- **S-Video** — luminance and chrominance on separate wires; better quality. 4-pin mini-DIN.
- **Component** — the signal split further (YPbPr, YCbCr); highest analog quality. Three RCA connectors.

**Television broadcast standards** were regional and incompatible: **NTSC** (North America, Japan; 525 lines, ~30 fps), **PAL** (most of Europe, Asia, Australia; 625 lines, 25 fps, more consistent colour), and **SECAM** (France, parts of Eastern Europe; a different colour-encoding method).

## Digital video

Digital video represents the signal as data, which brings higher quality, resistance to noise and generation loss, perfect copies, non-linear editing (jump to and rearrange any part), higher resolutions (HD, 4K, 8K), and interactivity.

A **codec** (coder-decoder) compresses video for storage and transmission and decompresses it for playback. The major ones:

| Codec | Used for |
| --- | --- |
| MPEG-1 | Video CD, early digital video |
| MPEG-2 | DVD, digital TV broadcast (DVB, ATSC) |
| MPEG-4 Part 2 (DivX, Xvid) | Early internet video |
| H.264 / AVC (MPEG-4 Part 10) | Blu-ray, streaming, HDTV — the workhorse |
| HEVC / H.265 | Roughly double H.264's efficiency; 4K and 8K |
| AV1 | Open, royalty-free; internet video, better than HEVC |

How these actually compress — spatial transform coding plus motion compensation between frames — is covered in [compression](/citadel/computer-graphics/compression); the streaming protocols that deliver them are in [multimedia architecture](/citadel/computer-graphics/architecture).

On a PC, the **GPU** has dedicated hardware for video decode and encode, offloading the CPU; playback software (VLC, Windows Media Player, QuickTime) uses codecs to handle the various formats.

## Video processing

1. **Pre-production** — planning, scripting, storyboarding.
2. **Production** — shooting the raw footage and sound.
3. **Post-production** — ingesting footage, non-linear editing (assembling clips, transitions), colour correction and grading, audio editing and mixing, visual effects and compositing, titles and motion graphics, and finally rendering to a delivery format.

Non-linear editors: Adobe Premiere Pro, DaVinci Resolve, Final Cut Pro, Avid Media Composer, HitFilm.

**Video file formats** are **containers** that hold codec-compressed video, audio, and metadata like subtitles:

- `.mp4` — very common, widely supported; usually H.264 or HEVC video with AAC audio.
- `.mov` — Apple's format, common in professional workflows.
- `.avi` — older Microsoft container.
- `.mkv` — open, flexible, popular for HD and 4K.
- `.webm` — open format for web video, VP9 or AV1 video with Opus or Vorbis audio.

## The one idea to keep

Audio and video are continuous signals turned into a stream of samples or frames, and every quality knob — sampling rate, bit depth, frame rate, resolution — is a choice about how finely to sample time and amplitude. On top of that, a codec removes what a human is least likely to notice, and a container bundles the compressed streams together so a player can find them.
