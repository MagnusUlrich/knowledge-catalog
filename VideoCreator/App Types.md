---
tags: [videocreator, linux, web, wine, chromium]
---

# App Types

## `linux` — Native Linux GUI

Any executable that opens a window in X11.

```bash
APP_TYPE=linux
APP_PATH=/apps/myapp          # or just a command name like "gedit"
```

- The command is split on spaces: `APP_PATH=gedit /apps/doc.txt` works.
- Place the binary in `./apps/` on the host (mounted as `/apps/` in container).
- GTK, Qt, Electron, wxWidgets all work as long as they use X11.

## `web` — Browser App (Chromium)

```bash
APP_TYPE=web
APP_PATH=file:///apps/index.html   # local file
# or
APP_PATH=http://localhost:3000     # service running on the host
```

Chromium is launched with:
- `--no-sandbox` (required inside Docker)
- `--disable-gpu` (no GPU in virtual display)
- `--window-size=1920,1080`

Extra flags via `CHROMIUM_FLAGS`:
```bash
CHROMIUM_FLAGS="--force-dark-mode --lang=de"
```

**Accessing host services:** use `host.docker.internal` as the hostname or pass `--network=host` to `docker run`.

## `wine` — Windows `.exe`

```bash
APP_TYPE=wine
APP_PATH=/apps/notepad++.exe
WINEPREFIX=/root/.wine          # default
```

- `winehq-stable` is installed in the image.
- First run initialises the Wine prefix automatically (may take ~30 s).
- For complex installers, consider pre-building the Wine prefix in a custom image layer.
- `WINEDEBUG=-all` is set automatically to suppress Wine debug noise from the recording.

### Tips for Wine apps

- Wait at least 6 s in `demo.run()` before interacting — Wine boot is slow.
- If the app needs a 32-bit runtime, it's already handled by `dpkg --add-architecture i386` in the Dockerfile.
- Winetricks packages can be added to the Dockerfile before the Wine install step.
