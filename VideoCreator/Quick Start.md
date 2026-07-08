---
tags: [videocreator, quickstart, docker]
---

# Quick Start

## 1. Build the image

```bash
cd C:\Users\ulrich\Documents\Dev\tools\VideoCreator
docker build -t videocreator .
```

The build downloads all packages (Xvfb, FFmpeg, Chromium, Wine, Python libs).  
After the image is built, the container runs **fully offline**.

## 2. Run the smoke test

Record `xeyes` (bundled in the image) for 10 seconds:

```bash
docker run --rm \
  -v "$(pwd)/output:/output" \
  -e APP_TYPE=linux \
  -e APP_PATH=xeyes \
  -e DURATION=10 \
  -e OUTPUT_NAME=xeyes_smoke \
  videocreator
```

Open `output/xeyes_smoke.mp4` — you should see the xeyes window for 10 s.

## 3. Record a real app via docker-compose

```bash
# 1. Copy the example env file
cp .env.example .env

# 2. Edit .env to point at your app
APP_TYPE=linux
APP_PATH=/apps/myapp
DEMO_SCRIPT=/demos/myapp_demo.py
OUTPUT_NAME=myapp_demo
DURATION=60

# 3. Place the app in ./apps/ and the demo script in ./demos/
# 4. Run
docker-compose up
```

The MP4 is written to `./output/myapp_demo.mp4`.

## 4. Write a custom demo

```python
# demos/myapp_demo.py
import time, pyautogui
from src.demos.base import BaseDemo

pyautogui.FAILSAFE = False

class Demo(BaseDemo):
    def run(self, app_pid: int) -> None:
        time.sleep(2)          # wait for app window
        pyautogui.click(960, 540)
        pyautogui.hotkey("ctrl", "n")
        time.sleep(1)
        pyautogui.typewrite("Hello VideoCreator!", interval=0.05)
        time.sleep(2)
```

See [[Demo Scripts]] for the full API.
