---
tags: [videocreator, demo, pyautogui, scripting]
---

# Demo Scripts

## BaseDemo API

```python
# src/demos/base.py
class BaseDemo:
    def setup(self) -> None:
        """Called before the app launches. Optional."""

    def run(self, app_pid: int) -> None:
        """
        Required. Recording is live while this runs.
        Return when the demo is done — recorder stops immediately after.
        """

    def teardown(self) -> None:
        """Called after the recording stops. Optional."""
```

## Writing a Script

1. Create a `.py` file anywhere (usually `./demos/` on the host).
2. Define `class Demo(BaseDemo)` and implement `run()`.
3. Pass the path via `DEMO_SCRIPT` env var or `--demo` flag.

```python
import time, pyautogui
from src.demos.base import BaseDemo

pyautogui.FAILSAFE = False   # required for headless operation
pyautogui.PAUSE = 0.05       # default pause between actions

class Demo(BaseDemo):
    def setup(self):
        pass  # optional

    def run(self, app_pid: int) -> None:
        time.sleep(2.5)               # let the app render
        pyautogui.click(960, 540)     # click screen centre
        pyautogui.hotkey("ctrl", "n") # keyboard shortcut
        pyautogui.typewrite("Hello!", interval=0.05)
        time.sleep(2)

    def teardown(self):
        pass  # optional
```

## Useful PyAutoGUI Calls

| Action | Code |
|--------|------|
| Click at position | `pyautogui.click(x, y)` |
| Right-click | `pyautogui.rightClick(x, y)` |
| Move mouse | `pyautogui.moveTo(x, y, duration=0.5)` |
| Type text | `pyautogui.typewrite("text", interval=0.05)` |
| Press key | `pyautogui.press("enter")` |
| Hotkey | `pyautogui.hotkey("ctrl", "s")` |
| Scroll | `pyautogui.scroll(5)` (positive = up) |
| Screen size | `pyautogui.size()` → `(width, height)` |

## Using xdotool (subprocess)

For window-level control (focus, resize, move):

```python
import subprocess

def focus_window(title_fragment: str) -> None:
    subprocess.run(["xdotool", "search", "--name", title_fragment,
                    "windowfocus", "--sync"], check=False)
```

## Duration vs Demo Exit

- If `DURATION > 0`, the demo is given that many seconds maximum; if `run()` returns before the timeout the recording stops immediately.
- If `DURATION=0`, only the demo's `run()` return controls when recording stops.

## Built-in Examples

| Script | Use case |
|--------|----------|
| `src/demos/default.py` | Fallback: idle for `DURATION` seconds |
| `src/demos/examples/gtk_demo.py` | Generic Linux GUI walkthrough |
| `src/demos/examples/web_demo.py` | Chromium web app tour |
| `src/demos/examples/wine_demo.py` | Windows `.exe` via Wine |
