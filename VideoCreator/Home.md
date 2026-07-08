---
tags: [videocreator, tooling, docker, recording]
---

# VideoCreator

A fully offline, Docker-based tool that launches any application, runs a scripted demo, and captures a **30 FPS H.264 MP4**.

## Supported App Types

| Type | How it runs |
|------|-------------|
| `linux` | Native Linux GUI app via X11 (GTK, Qt, Electron, …) |
| `web` | Web app opened in bundled Chromium |
| `wine` | Windows `.exe` via Wine (winehq-stable) |

## Key Design Decisions

- **Xvfb** provides a headless X11 display — no physical screen or GPU required.
- **FFmpeg x11grab** captures the virtual display at exactly 30 FPS into H.264/MP4.
- **Demo scripts** are plain Python files that subclass `BaseDemo` — scriptable, reviewable, reproducible.
- Everything is installed at image build time — **zero network calls at runtime**.

## Quick Links

- [[Quick Start]] — get a video in 5 minutes
- [[Architecture]] — pipeline overview
- [[Demo Scripts]] — how to write your own
- [[App Types]] — linux / web / wine details
- [[Troubleshooting]] — common issues

## Source Location

```
C:\Users\ulrich\Documents\Dev\tools\VideoCreator\
```
