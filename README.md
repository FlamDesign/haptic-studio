# Haptics Studio

A browser-based tool for generating Apple Core Haptics (`.ahap`) files from audio. Built for the Flam design team to create haptic tracks for video projects.

---

## How it works

You render a purpose-built audio control track from Premiere (MP3 works well) — placing sound effects and tones wherever you want haptic feedback, with volume reflecting the desired intensity. Drop that file into Haptics Studio and it analyses the audio to generate a structured `.ahap` file ready to use in your iOS project.

The tool runs entirely in the browser. No uploads, no server, no account needed.

---

## Workflow

1. Render your haptic control track from Premiere as MP3
2. Drop the file into Haptics Studio
3. Click **Analyze** — the tool runs a 3-band frequency analysis and generates events
4. Review the timeline — scrub, zoom, and adjust parameters if needed
5. Download the `.ahap` file

---

## Analysis

The tool splits audio into three frequency bands before detecting events:

| Band | Range | Maps to |
|------|-------|---------|
| Sub-bass | 20–200 Hz | Continuous beds (rumble, sustained haptics) |
| Mids | 200–2kHz | Music transients (beats, rhythmic hits) |
| Highs | 2kHz+ | SFX transients (impacts, sharp hits) |

Sharpness values in the AHAP output are derived automatically from the high/sub frequency ratio — so percussive high-frequency sounds get high sharpness, low rumbles get low sharpness.

**Adaptive thresholding** — the detection threshold floats with a rolling window average of local RMS energy. This means quiet sections and loud sections are treated independently, so a low-intensity moment in your track generates events at the right density without being drowned out by louder sections.

**Bed fades** — each continuous bed event gets its own `HapticIntensityControl` parameter curve, sampled from the audio envelope at that region. The result is beds that naturally swell and fade with the source audio rather than cutting in and out at a flat intensity.

---

## Timeline

- **Pinch** to zoom (MacBook trackpad)
- **Two-finger swipe** to pan
- **Click the ruler** to seek, **drag** to scrub (like Premiere)
- **Drag events** to reposition
- **Double-click** an event to delete
- **Z** to undo

---

## Parameters

### Detection
| Parameter | What it does |
|-----------|-------------|
| Adaptive window | Rolling window size for local RMS normalisation. Larger = smoother baseline. |
| Transient sensitivity | How aggressively hits are detected relative to local baseline |
| Min hit spacing | Minimum gap between transient events in ms |
| SFX sensitivity | Detection threshold for high-frequency onset events |
| Bed threshold | Minimum local energy to form a continuous bed region |
| Min bed duration | Rejects beds shorter than this |
| Curve resolution | Sample interval for the global intensity curve |

### Shaping
| Parameter | What it does |
|-----------|-------------|
| Global intensity | Scales all event intensities at export |
| Transient / SFX / Bed intensity | Per-type intensity multipliers |
| Bed fade points | Number of envelope samples per bed for the fade curve |

---

## Output format

The generated `.ahap` follows Apple's Core Haptics pattern format:

- `HapticTransient` events for beats and SFX hits
- `HapticContinuous` events for sustained beds
- `HapticIntensityControl` parameter curves — one global curve tracking overall energy, plus per-bed curves for fade in/out shapes

Load the `.ahap` directly into your iOS project using `CHHapticEngine`.

---

## Browser support

Works in Chrome and Safari. Audio decoding requires the file to have a decodable audio track — MP3 and AAC (MP4) work reliably in both browsers.
