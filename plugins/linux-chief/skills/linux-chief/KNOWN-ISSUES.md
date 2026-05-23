# Known Issues and Workarounds

Current bugs and limitations with documented workarounds.

## KDE Plasma 6

### GTK Apps Don't Follow Theme on Wayland
**Status:** Won't fix (use X11)
**Workaround:** Use X11 session instead of Wayland for full GTK theme compatibility.

### Keyboard Layout Switching Unreliable
**Status:** Workaround available
**Workaround:** See `docs/kde-keyboard-layout-switching-fix.md`

## PipeWire

### "Too Many Connections" After Long Uptime
**Status:** Known issue
**Cause:** Apps don't properly release PipeWire connections over time.
**Workaround:** Use `/fix-audio` skill. Reboot every 3-4 days for prevention.

### Audio Crackling in Electron Apps (Discord, Vesktop)
**Status:** Partially fixed in newer versions
**Workaround:**
- Increase audio buffer: `pw-metadata -n settings 0 clock.force-quantum 1024`
- Use Legacy audio subsystem in Discord settings

## Docker

### Overlay Mounts Interfere with KDE
**Status:** Documented
**Workaround:** See `docs/docker-kde-overlay-mounts-fix.md`

## DaVinci Resolve

### GPU Memory Issues with Large Projects
**Status:** Hardware limitation
**Workaround:**
- Close other GPU-intensive apps (ComfyUI, browsers with hardware acceleration)
- Use proxy media for editing, switch to full quality for final render

### Some Codecs Not Supported (Free Version)
**Status:** By design
**Workaround:** Convert files first using `docs/dolphin-convert-to-resolve.md`

## ComfyUI

### ControlNet Crash on Large Images
**Status:** Memory issue
**Workaround:** See `docs/comfyui/z-image-controlnet-crash-fix.md`

## Claude Code / Dev Tools

### Orphaned Processes After Swarm/dev-maestro
**Status:** Known behavior
**Cause:** Claude Code swarm agents don't always clean up when parent dies.
**Workaround:** Use `/process-cleanup` skill regularly, especially after using dev-maestro.

### Vitest Workers Persist After Test Run
**Status:** Known issue
**Cause:** Vitest workers may not terminate properly on certain test failures.
**Workaround:**
```bash
pkill -f "vitest"
```

### earlyoom Killing Claude Sessions
**Status:** Expected when RAM exhausted
**Cause:** Claude Code sessions can grow to 400-600MB each; multiple sessions fill RAM.
**Workaround:** Use `/process-cleanup` to clean stale sessions before running new tasks.

## System

### Google Antigravity Service Outage (Jan 2026)
**Status:** External issue
**Details:** See `docs/google-antigravity-outage-jan-2026.md`

---

*Last updated: 2026-01-24*
