---
tags: [videocreator, architecture, docker, xvfb, ffmpeg]
---

# Architecture

## Recording Pipeline

```
entrypoint.sh
  ├─ Xvfb :99  (virtual X11 display, 1920×1080×24)
  ├─ Openbox   (lightweight window manager)
  └─ python src/main.py
        ├─ display.py   → wait_for_display() confirms Xvfb is ready
        ├─ recorder.py  → ffmpeg -f x11grab -framerate 30 → libx264 MP4
        ├─ runner.py    → launches app (linux / chromium / wine)
        ├─ demo.run()   → scripted interactions via PyAutoGUI / xdotool
        ├─ recorder.stop()  → ffmpeg SIGINT → file finalised
        └─ /output/<name>.mp4
```

## Component Responsibilities

| File | Role |
|------|------|
| `scripts/entrypoint.sh` | Boot Xvfb + Openbox, then hand off to Python |
| `src/display.py` | Poll `xdpyinfo` until display is alive; set background colour |
| `src/recorder.py` | Wrap FFmpeg process; `start()` / `stop()` / context manager |
| `src/runner.py` | Launch the target app with correct env; `AppRunner` class |
| `src/demos/base.py` | `BaseDemo` abstract class (`setup / run / teardown`) |
| `src/demos/default.py` | Fallback: just wait for `DURATION` seconds |
| `src/main.py` | CLI (`click`): wire display → recorder → runner → demo |

## Volume Mounts

| Host path | Container path | Purpose |
|-----------|----------------|---------|
| `./output` | `/output` | MP4 files written here |
| `./apps` | `/apps` | Apps / installers to record |
| `./demos` | `/demos` | Custom demo scripts |

## FFmpeg Command

```bash
ffmpeg -y \
  -f x11grab -framerate 30 -video_size 1920x1080 -i :99.0 \
  -c:v libx264 -preset fast -crf 18 \
  -pix_fmt yuv420p -movflags +faststart \
  /output/demo.mp4
```

- `-crf 18` → near-lossless quality; raise to 23–28 for smaller files
- `-movflags +faststart` → MP4 header at the front (better streaming)
