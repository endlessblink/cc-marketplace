---
name: reliable-cursor-overlay
description: Use when building screen recording cursor capture, cursor telemetry, playback preview overlays, or FFmpeg/export cursor rendering. Applies to Linux/X11 screen recorders, Screen Studio-style exports, and any workflow where cursor position must stay synced through crop/scale/zoom transforms.
---

# Reliable Cursor Overlay

Use this workflow when implementing cursor rendering for screen recordings.

## Core Rule

Do not render cursor movement by chaining one FFmpeg `overlay` filter per telemetry sample directly onto the source video. It becomes fragile, slow, hard to sync, and easy to break when crop/scale/zoom changes.

Use a separate cursor layer and a shared coordinate transform.

## Recommended Architecture

1. Record screen without the system cursor.
   - For FFmpeg `x11grab`, use `-draw_mouse 0`.
   - This avoids double cursors and allows consistent styled cursor rendering.
2. Capture cursor telemetry alongside recording.
   - Store `frame`, `timeMs`, `x`, `y`, `type`, and button/click state when available.
   - Coordinates must be normalized into source-recording pixels.
3. Generate a transparent cursor video layer from telemetry.
   - Same duration, fps, and source dimensions as the recording.
   - Render a real cursor asset, not a text glyph.
   - Interpolate between samples for smooth movement.
4. Apply visual transforms once.
   - Define a single `source -> styled canvas` transform for fit/crop/zoom.
   - Apply that transform to both video and cursor layer.
5. Overlay the cursor layer once during export.
   - Use FFmpeg `overlay` with timestamp-normalized inputs.
   - Add `setpts=PTS-STARTPTS` on overlay inputs when combining separate streams/files.
6. Use the same transform for preview.
   - DOM/canvas preview cursor should be driven from the same telemetry and transform as export.

## Coordinate Notes

- Electron `screen.getCursorScreenPoint()` returns DIP coordinates, not physical pixels.
- On Linux/X11, convert DIP to physical recording coordinates using display bounds and scale factor.
- FFmpeg `x11grab` input syntax is `[host]:display.screen+x_offset,y_offset`; offsets are relative to the X11 screen top-left.
- `x11grab` `video_size` defines the captured region. Cursor coordinates outside that region are valid and must be **passed through, not clamped**.

## Critical: Electron `screen.getCursorScreenPoint()` is broken on Linux/X11 multi-monitor (v29+)

Documented regression in Electron v29 and later: `screen.getCursorScreenPoint()` returns stale/stuck values when the cursor leaves the primary display. See [electron/electron#42519](https://github.com/electron/electron/issues/42519) and [#41496](https://github.com/electron/electron/issues/41496). Symptom: cursor moves correctly on the recorded display, then stops updating once it crosses to a secondary monitor, and never resumes — creating "cursor stuck after returning from second monitor" in any export or preview that consumes the recorded telemetry.

**Use a different cursor source on Linux**:

- **xdotool** (recommended for X11 today): spawn `xdotool getmouselocation --shell` per sample tick, parse with `/X=(-?\d+)\s+Y=(-?\d+)/`. ~1 ms per call; fine at 100 ms sampling. Returns negative x for cursors on left-side monitors. Standard Linux package, no JS deps.
- **node-x11**: in-process X server query. Faster than spawn but adds a native binding.
- **Fallback** to `screen.getCursorScreenPoint()` when xdotool/node-x11 isn't available so single-monitor setups still work.

Wrap the Linux source in dependency-injected `getCursorPoint` so the recorder is testable without touching real X11.

## Wayland is fundamentally different

Wayland deliberately blocks apps from querying global cursor position for security. There is no portable `getCursorScreenPoint()` equivalent. Rough rules:

- Workarounds via `wl-find-cursor` (layer-shell + virtual-pointer protocols) work on wlroots compositors (Sway, Hyprland) but **GNOME refuses** to implement those protocols.
- The architecturally correct Wayland path is to abandon "track cursor in app" entirely. Switch screen capture to **xdg-desktop-portal + PipeWire ScreenCast**: the compositor itself draws the cursor into the captured stream. Recording arrives with cursor pre-rendered. No telemetry, no overlay, no zoom-aware cursor scaling — the cursor IS just pixels in the source video.
- This is a big architectural pivot for any project built around the "transparent cursor layer" pattern. Plan it as its own task.

## Critical: Do Not Clamp Cursor Coordinates

When a user moves the cursor onto a second monitor during recording, its coordinates exceed the recorded screen bounds. **Do not clamp these positions** at any layer:

- **Recorder layer**: store the raw absolute coordinates after offset/DPI scale conversion. Do not clamp to `[0, width-1]`. Multi-monitor users need the off-screen positions preserved so the cursor disappears off the edge cleanly during playback rather than sticking at the visible bound.
- **Render layer**: when burning the cursor into export frames or drawing it in preview, pass off-screen coordinates through. ASS subtitle renderers and Canvas2D both clip naturally past their bounds — the cursor just doesn't draw when its anchor is off the visible canvas.

Symptom of incorrect clamping: cursor appears "stuck on a vertical line" at the screen edge instead of leaving frame as expected when crossing to another monitor.

## Styling Notes

- Keep source recording full-frame until explicit zoom/viewport logic exists.
- Do not crop the source video just to make it look zoomed if cursor overlay is source-coordinate based.
- When adding zooms later, model zoom as a transform and apply it to video and cursor together.
- Use a real cursor asset with known hotspot. If the asset lacks metadata, document the assumed hotspot.

## FFmpeg Notes

- `overlay` evaluates `x` and `y` per frame by default (`eval=frame`).
- Variables `n` and `t` are only available for per-frame evaluation.
- If overlay inputs have different initial timestamps, normalize them with `setpts=PTS-STARTPTS`.
- Chaining many overlays should be avoided; FFmpeg docs explicitly recommend testing efficiency when chaining overlays.

### `nullsrc` and `zoompan` default to 25 fps — pin them

Both `nullsrc` (used to generate styled-canvas backgrounds via `geq`) and `zoompan` (used for time-varying crop/scale) default to 25 fps regardless of source rate. In a styled-export filter graph where these are inputs, the entire output gets emitted at 25 fps even when the source recording is 30/60 fps. Symptom: the final MP4 plays slower than the recording, audio drifts, and cursor positions baked at source PTS appear off by frame.

Fix: pin both to the source fps explicitly.

```
nullsrc=s=${width}x${height}:r=${fps},format=rgb24,...[bg]
zoompan=z='...':x='...':y='...':d=1:s=${sourceW}x${sourceH}:fps=${fps}
```

Lock this with a smoke test that runs `ffprobe -show_entries stream=r_frame_rate` against the styled export and asserts it equals the source fps.

### `zoompan` expression syntax quirks

When generating per-frame zoom expressions for `zoompan`:

- Use `on` (output frame counter), **not** `n`. `n` is undefined inside `zoompan` expressions.
- `between(n, start, end)` is **not** supported. Use nested `if(lt(on, X), ..., if(lt(on, Y), ..., ...))`.
- `pow(t, k)` works for polynomial easing (e.g., Ken Perlin smootherStep `6*pow(t,5)-15*pow(t,4)+10*pow(t,3)`).
- The current frame's zoom value is exposed as `zoom` inside the `x` and `y` expressions, so you can derive crop window position from focal point + current scale: `iw * max(0, min(focalX - 1/(2*zoom), 1 - 1/zoom))`.
- Commas inside single-quoted expressions don't need escaping when filter strings reach FFmpeg via `spawn` (no shell), but they DO need `\,` escaping at the shell-CLI layer.

### ASS subtitles as a cursor layer

A workable alternative to a separate transparent video layer: render the cursor stream as an ASS subtitle file, burn it into the source frames with the `subtitles=` filter before any zoom or styling.

```
[0:v]setpts=PTS-STARTPTS[base]
[base]subtitles=cursor.ass[with_cursor]
[with_cursor]<crop+scale or zoompan>,...[screen]
```

ASS supports `\move(x1, y1, x2, y2, t1, t2)` for smooth cursor interpolation and `\p1`/`\p0` for vector path drawing. Use a small polygon for the cursor shape (e.g., the standard arrow `m 0 0 l 0 26 l 7 20 l 12 33 l 18 31 l 13 19 l 24 19 l 0 0`) — this draws as vector, stays crisp under any zoom downstream.

Cursor coordinates passed to ASS must be in source pixels and unclamped (see "Critical: Do Not Clamp Cursor Coordinates").

## Cursor + Zoom Interaction

When the export pipeline applies zoom to the source, the cursor should scale with the source pixels. Achieve this by burning the cursor into the source frame BEFORE the zoom filter runs, so the cursor pixels are part of what gets cropped and scaled by zoompan.

Industry pattern (validated against Screen Studio and FocuSee):
- Cursor scales with zoom (preserves visual parity).
- Cursor is rendered as a vector path so it stays crisp at any zoom level (no pixelation).
- Constant-size cursor would diverge from the export and is the wrong tradeoff for screen-recording tools.

In a Canvas2D preview that mirrors the export, draw the cursor polygon INSIDE the zoom transform (`ctx.save() → ctx.translate → ctx.scale → drawImage(video) → drawCursor → ctx.restore()`). The cursor scales naturally with the canvas matrix.

### Cursor-follow zoom phase model

Professional screen-recording apps separate the action zoom target from cursor-follow polish:

- **Ramp-in** should target the marker/action focal point, usually the click or auto-zoom trigger. Do not chase live cursor movement while scale is changing; it creates wobble and makes the zoom feel unintentional.
- **Hold** may follow the cursor with smoothing, but must apply a final containment correction so the current cursor remains inside the visible zoom window through the last hold frame (`holdEnd - 1`). Smoothing alone can lag too far behind fast cursor moves.
- **Ramp-out** should start from the final contained hold focal, then ignore later cursor movement and ease/reveal back toward center as scale returns to 1. Continuing to chase the cursor during zoom-out makes the exit feel nervous and can hide the actual reveal.

Keep these phases in one shared resolver used by both preview and export. Lock it with tests for: ramp-in ignores cursor drift, final hold frame contains the cursor, zoom-out starts from the final contained hold focal, and cursor movement after `holdEnd` does not change the zoom-out crop.

## Preview/Export Parity via Shared Resolver

For projects with both a styled export pipeline and a renderer preview, route both sides through a single shared frame-resolution function (e.g. `resolveFrame(project, frame)`) that returns a complete render description (`{ cameraTransform, cursor, layers, ... }`).

- Math (zoom transform, easing, focal-point clamping) lives in one shared package.
- Preview consumes `cameraTransform` to drive Canvas2D `ctx.translate + ctx.scale`.
- Export consumes the same transform to derive `zoompan` expressions or per-frame filter parameters.

This keeps preview and export pixel-aligned automatically when the math changes, instead of needing to keep two implementations in sync.

## Verification Checklist

- Export with no cursor telemetry succeeds.
- Export with cursor telemetry succeeds.
- Cursor is visible at corners and edges, not hidden by crop/scale.
- Cursor follows the same point in source content before and after styled scaling.
- Movement is smooth on a diagonal path, not stair-stepped.
- Preview and exported MP4 agree visually.
- Fresh real recording is tested, not only a synthetic fixture.
- **Multi-monitor test**: cursor moved onto a second monitor disappears off the recorded edge cleanly, then reappears on return — does not stick at the visible bound.
- **fps check**: `ffprobe stream=r_frame_rate` on the styled export equals the source fps.
- **Zoom + cursor**: at the marker hold phase, cursor is positioned correctly in the zoomed view and visibly scales with the source pixels (not constant on-screen size).
