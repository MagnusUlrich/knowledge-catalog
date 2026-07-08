---
tags: [videocreator, troubleshooting, xvfb, wine, ffmpeg]
---

# Troubleshooting

## Xvfb / Display

**Symptom:** `Cannot open display :99` or app exits immediately.

- The entrypoint polls `xdpyinfo` for up to 10 s. If the container starts but Xvfb fails, check `docker logs <container>` for Xvfb errors.
- Check that the container is not running as a completely locked-down user — Xvfb needs `/tmp/.X11-unix`.
- Add `-e DISPLAY=:99` explicitly if overriding the entrypoint.

**Symptom:** Black frames in the output MP4.

- The app window may not have appeared before recording started. Increase the `time.sleep()` at the start of `demo.run()`.
- Check that the window manager (Openbox) started successfully.

## FFmpeg / Recording

**Symptom:** `Output file /output/demo.mp4 not found` after the run.

- The `/output` volume might not be mounted. Check `docker run -v $(pwd)/output:/output`.
- FFmpeg may have crashed — run with `docker run ... 2>&1 | grep ffmpeg` to see its stderr.

**Symptom:** MP4 plays back choppy / not 30 FPS.

- The container may be CPU-constrained. Lower `-crf` (raises quality/CPU) or use `-preset ultrafast`.
- Alternatively, record at lower resolution and upscale: `RESOLUTION=1280x720`.

**Symptom:** MP4 file is corrupted / unplayable.

- FFmpeg was killed mid-recording without finalising. Make sure `docker stop` (not `docker kill`) is used, giving the container time for graceful shutdown via SIGINT.

## Chromium (web apps)

**Symptom:** Chromium exits with `Segmentation fault`.

- Increase shared memory: add `--shm-size=2gb` to `docker run` or check `shm_size: '2gb'` in `docker-compose.yml`.
- Add `--disable-dev-shm-usage` to `CHROMIUM_FLAGS`.

**Symptom:** Page renders blank / white.

- If targeting `localhost`, use `--network=host` so the container can reach host services.
- For local HTML files use `file:///apps/index.html` not a relative path.

## Wine (Windows apps)

**Symptom:** Wine prefix initialises every run, taking 30+ s.

- Pre-initialise the prefix in the Dockerfile: `RUN DISPLAY=:99 ... wineboot --init`.
- Or mount a pre-built Wine prefix from the host: `-v ./wineprefix:/root/.wine`.

**Symptom:** App window does not appear / Wine crashes.

- Set `WINEDEBUG=+all` temporarily and review `docker logs` for the error.
- Some apps require Winetricks packages (`corefonts`, `dotnet48`, etc.) — add to Dockerfile.

**Symptom:** Chinese/Cyrillic/special characters show as boxes.

- Add `fonts-noto` (or other font packages) to the Dockerfile apt install list.

## General

**Run a debug shell inside the container:**

```bash
docker run --rm -it --entrypoint bash \
  -v $(pwd)/output:/output \
  videocreator
# inside: DISPLAY=:99 xeyes &  to verify the display
```

**Check the virtual display manually:**

```bash
DISPLAY=:99 xdpyinfo | grep dimensions
```
