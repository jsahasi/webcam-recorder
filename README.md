# Webcam Studio

A zero-install, single-file web app for recording high-quality clips — webcam or
screen share, with an optional speech-following teleprompter.
Everything is local — no uploads, no accounts, no build step.

## Run it

**Easiest:** double-click `index.html` (works in Chrome/Edge/Firefox).

**Most reliable (any browser, incl. Safari):** serve it over localhost:

```bash
cd webcam-recorder
python3 -m http.server 8000
# open http://localhost:8000
```

On first run the browser asks for camera + microphone permission — allow it.
(On macOS, your browser also needs permission under
*System Settings → Privacy & Security → Camera / Microphone*. Screen sharing
may prompt for *Screen Recording* permission too.)

## Features

- **Sources**: Webcam, or **Screen share** (screen/window/tab). In screen mode the
  webcam can be toggled off entirely, or kept as a **picture-in-picture overlay**
  in any corner — composited live, so what you preview is exactly what's recorded.
- **Teleprompter**: paste a script, get an overlay near the camera. It **listens to
  your speech and highlights the next N words** (configurable, default 5), dimming
  what you've already said and **underlining the last word it heard** so you can see
  exactly where the tracker is. Two follow modes:
  - *Follow my speech* — Web Speech API tracks where you are in the script
  - *Timed scroll* — steady pace, adjustable 60–220 wpm
  - **Pace, font size, highlight count and mode are adjustable live, mid-recording**
    (slider or `↑`/`↓` keys). Browsers without speech recognition (Firefox) fall
    back to timed scroll automatically.
- **Watermark**: a small credit line burned into the bottom of the video —
  defaults to `Webcam recording Agent: copyright Sahasi <current year>` (year is
  automatic). Edit it to anything, or clear the field for no watermark. When set,
  the preview shows exactly what the file will look like, in all modes (webcam,
  screen, screen+PiP).
- **Persistence**: every setting — devices, resolution, fps, quality, countdown,
  overlay options, watermark text, teleprompter on/off, **the script itself**,
  pace — is saved to localStorage and restored on next visit.
- **Recording**: MP4 (H.264/AAC) where supported, else WebM (VP9/Opus);
  up to 4K webcam or native screen resolution; 24/30/60 fps; ~8/14/24 Mbps
  presets scaled by resolution; 192–256 kbps audio; pause/resume; countdown;
  mic-level meter; live time + file size; timestamped downloads.

## Use it

1. Pick a **Source**: *Webcam* or *Screen share* (browser asks what to share).
   In screen mode, use **Include webcam overlay** to toggle your camera off or
   add it as a corner inset.
2. Optionally enable the **Teleprompter**, paste your script, choose a follow
   mode and pace.
3. Check the **mic level** meter moves when you speak.
4. Hit **● Record** (or `Space`). Pause/resume anytime; tweak prompter pace live.
5. Hit **■ Stop** → review → **⬇ Download clip** (saved to Downloads as
   `webcam-YYYYMMDD-HHMMSS.mp4|webm`). Not happy? **↺ Re-record**.

## Notes & limits

- **Windows PCs: fully supported.** Chrome or Edge on Windows 10/11 runs everything —
  webcam, screen share, teleprompter with speech follow (internet required), MP4 output.
  In one way Windows is *better* than macOS: Chrome on Windows can capture **system
  audio when sharing the whole screen**, not just tab audio. Firefox on Windows works
  too, with timed-scroll prompter and WebM output. For the localhost method on Windows
  use `py -m http.server 8000` (or `python -m http.server 8000`).
- **Screen audio on macOS**: browsers can only capture audio when you share a
  *browser tab* (Chrome/Edge: "Also share tab audio"). Sharing a window or the
  whole screen has no system audio on macOS — your mic is always recorded regardless.
- **Speech follow needs internet on Chrome/Edge** (audio is processed by the
  browser vendor's speech service). Safari uses its own service. Offline or
  Firefox → the app switches to timed scroll automatically.
- The speech recognizer listens on the system **default** microphone, not the
  one picked in settings.
- The teleprompter overlay is **never baked into webcam recordings** (it's a DOM
  overlay). But if you screen-share the window/screen showing this app, whatever
  is visible there — prompter included — will be captured. Share a different
  window/tab if you want a clean recording.
- **Watermark + hidden tabs**: burning in a watermark means frames are drawn
  through a canvas, and browsers throttle canvas painting in background tabs.
  Keep this tab visible while recording with a watermark; without one, recording
  continues fine even if you switch away. Mirror preview is disabled while a
  watermark is set (the preview must match the file, text included).
- WebM files don't play in QuickTime; convert with
  `ffmpeg -i clip.webm -c:v libx264 -crf 18 -c:a aac clip.mp4`.

## Troubleshooting

| Symptom | Fix |
|---|---|
| Black preview / "permission denied" | Allow camera in the address bar; check macOS Privacy settings above |
| "Camera is in use by another app" | Quit Zoom/FaceTime/Photo Booth and retry |
| Safari: camera never starts on `file://` | Use the `python3 -m http.server` method |
| Screen picker cancelled / nothing happens | Pick **Screen share** again and choose a screen, window or tab |
| Screen share stops when I click the browser's "Stop sharing" bar | Expected — the app returns to webcam mode; re-select Screen share |
| Teleprompter doesn't follow my voice | Chrome needs internet for speech recognition; check the Teleprompter status line — it falls back to timed scroll |
| 4K option does nothing | Camera tops out below 4K — the actual negotiated resolution is shown under the preview |
| Want to wipe saved settings/script | DevTools → Application → Local Storage, or run `localStorage.clear()` in the console |
