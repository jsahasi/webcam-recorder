# Webcam Studio

A zero-install, single-file web app for recording high-quality **short clips
(under ~10 minutes)** — webcam or screen share, with an optional speech-following
teleprompter. Everything is local — no uploads, no accounts, no build step.

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

- **Cockpit UI**: the default view is just the monitor, the transport, and a row
  of icon toggles — source (webcam/screen), mic mute, teleprompter, auto-cut,
  settings. Teleprompter and capture settings live in collapsible panels (click
  the gear icon or the panel headers) — and since every setting persists, setup
  is a one-time thing.
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
  automatic). Clear the field for no watermark. When set, the preview shows
  exactly what the file will look like, in all modes (webcam, screen, screen+PiP).
  The field is read-only by design; to edit it, **click the word "Watermark"
  above the field** to unlock it (required for the free license's attribution
  terms to stay intact by default — see [License](#license)).
- **Persistence**: every setting — devices, resolution, fps, quality, countdown,
  overlay options, watermark text, teleprompter on/off, **the script itself**,
  pace — is saved to localStorage and restored on next visit.
- **Auto-cut silences**: optional live trimming — with the scissors toggle on,
  the recorder **auto-pauses after a sustained silence** (1–3 s, configurable in
  Capture settings) and **resumes the instant you speak**, so dead air never
  lands in the file. Same jump-cut result as post-hoc silence removers, with no
  re-encode. Inert while the mic is muted.
- **Recording**: MP4 (H.264/AAC) where supported, else WebM (VP9/Opus);
  up to 4K webcam or native screen resolution; 24/30/60 fps; ~8/14/24 Mbps
  presets scaled by resolution; 192–256 kbps audio; pause/resume; countdown;
  analog VU mic meter; live time + file size; timestamped downloads.

## Use it

1. Pick a source on the **icon row** under the transport: **webcam** or
   **screen share** (the browser asks what to share). Screen options — webcam
   overlay on/off, overlay corner, tab audio — are in **Capture settings**
   (gear icon).
2. Optionally toggle the **teleprompter** (script icon). If no script is saved
   yet its panel opens automatically — paste your script, pick a follow mode
   and pace.
3. Speak: the **VU needle** on the monitor should swing. Mute anytime with the
   mic icon.
4. Hit **Record** (or `Space`). **Pause/resume** anytime with the ⏸ button;
   toggle the **scissors icon** to auto-cut extended silences as you record;
   tweak prompter pace live with the slider or `↑`/`↓`.
5. Hit **Stop** → review → **Download clip** (saved to Downloads as
   `webcam-YYYYMMDD-HHMMSS.mp4|webm`). Not happy? **Re-record**.

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
| Can't find a setting (resolution, watermark, …) | Panels are collapsed by default — click the **gear icon**, or the **Capture settings** / **Teleprompter** headers |
| Recording "pauses" by itself mid-take | The **scissors toggle** (auto-cut) is on — it pauses the recorder during extended silences; the badge dot turns amber. Toggle it off to record continuously |
| Auto-cut never kicks in | Inert while the mic is **muted**; otherwise check the VU needle moves when you speak — a very quiet mic may sit below the silence threshold |
| Black preview / "permission denied" | Allow camera in the address bar; check macOS Privacy settings above |
| "Camera is in use by another app" | Quit Zoom/FaceTime/Photo Booth and retry |
| Safari: camera never starts on `file://` | Use the `python3 -m http.server` method |
| Screen picker cancelled / nothing happens | Pick **Screen share** again and choose a screen, window or tab |
| Screen share stops when I click the browser's "Stop sharing" bar | Expected — the app returns to webcam mode; re-select Screen share |
| Teleprompter doesn't follow my voice | Chrome needs internet for speech recognition; check the Teleprompter status line — it falls back to timed scroll |
| 4K option does nothing | Camera tops out below 4K — the actual negotiated resolution is shown under the preview |
| Want to wipe saved settings/script | DevTools → Application → Local Storage, or run `localStorage.clear()` in the console |
| Can't type in the watermark field | It's read-only by default — click the word **"Watermark"** above the field to unlock editing |

## License

Dual-licensed — see [LICENSE](LICENSE):

- **Free Attribution License**: free to use, copy, and modify, but the copyright
  notice and the built-in watermark (with its default attribution text) must be
  retained in all copies, deployments, and recordings. No selling.
- **Commercial License** (paid): removes the watermark/attribution requirement
  and permits sale and sublicensing. Contact the copyright holder.
