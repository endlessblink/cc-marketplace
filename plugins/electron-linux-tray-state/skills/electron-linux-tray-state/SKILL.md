---
name: electron-linux-tray-state
description: Use when fixing Electron system tray/status indicator behavior on Linux, especially stale menus, wrong right-click labels, missing icon updates, KDE/GNOME StatusNotifierItem quirks, recording indicators, or tray lifecycle bugs.
---

# Electron Linux Tray State

Use this skill for Electron tray/status-indicator issues on Linux, especially KDE/GNOME bugs where a tray icon or context menu shows stale state after app code updated or destroyed the tray.

## Core Facts

- Electron trays on Linux use StatusNotifierItem over DBus when available, with fallbacks depending on the desktop environment.
- Linux tray hosts may cache menu/icon/tooltip state outside the Electron `Tray` object.
- Electron docs explicitly require calling `tray.setContextMenu(...)` again after any menu change on Linux. Mutating `MenuItem`s is not enough.
- `destroy()` removes Electron's tray object, but it is not a reliable state transition for the desktop shell. A shell may briefly or persistently serve the last exported menu.
- Use PNG/JPEG icons or file paths for cross-platform reliability. SVG data URLs are less reliable on Linux tray hosts.
- Left-click/right-click activation varies by desktop environment because StatusNotifierItem does not standardize which user action counts as activation.

## Reliable Pattern

Prefer a persistent tray state machine over destroy/recreate as normal UI state.

States should be explicit and terminal states should be non-transient:

- `recording`: active red icon, menu contains recording controls.
- `finalizing`: amber icon, disabled menu says finalizing/saving.
- `saved` or `editor`: neutral/green icon, disabled menu says recording saved/editor open.
- `canceling`: neutral/gray icon, disabled menu says canceling/discarding.
- `discarded`: neutral/gray icon, disabled menu says recording discarded/ready.
- `idle`: optional app-level neutral tray if the product wants a persistent tray outside recording.

On every transition:

```js
tray.setImage(iconForState(state));
tray.setToolTip(tooltipForState(state));
tray.setContextMenu(Menu.buildFromTemplate(menuForState(state)));
```

Do not mutate old menu objects in place. Always build a fresh menu and pass it to `setContextMenu`.

## Anti-Patterns

- Do not use `tray.destroy()` as the primary way to mean "not recording anymore".
- Do not set `Finalizing...` and then immediately destroy the tray; Linux shell caches can keep that stale menu visible.
- Do not use transparent tray icons to hide an item; some Linux panels leave blank slots.
- Do not assume `right-click` events exist on Linux; Electron documents `right-click` as macOS/Windows only.
- Do not rely on SVG data URLs for tray icons if users report missing icons.

## Fix Workflow

1. Inspect all tray lifecycle functions and callers.
2. Identify every async transition: start, stop, restart, cancel, finalize success, finalize failure, app quit.
3. Replace destroy/recreate transitions with `updateTrayState(state)`.
4. Keep one `Tray` instance alive after stop in a stable terminal state (`saved`, `discarded`, or `idle`).
5. Destroy only on app quit, or only after setting a stable terminal state and confirming the product accepts possible shell caching.
6. Ensure every state calls `setImage`, `setToolTip`, and `setContextMenu` with fresh data.
7. Add or update tests for pure menu/state builders when possible.
8. Manually test on the target Linux desktop environment; automated Electron smoke tests usually cannot verify shell tray menu caching.

## Verification Checklist

- While active, menu says the active state, e.g. `Recording...`.
- Immediately after stop, menu says `Finalizing...`, not `Recording...`.
- After finalization and editor handoff, menu says `Recording saved` or `Editor is open`, not `Finalizing...`.
- Cancel lands on `Recording discarded`, not `Recording...`.
- Restart returns to the active `Recording...` state.
- App quit destroys the tray.

## References

- Electron Tray API: `https://www.electronjs.org/docs/latest/api/tray`
- Electron Linux tray DBus reimplementation PR: `https://github.com/electron/electron/pull/36333`
- Electron Linux tray click/menu issue: `https://github.com/electron/electron/issues/14941`
- Electron Linux tray icon regression: `https://github.com/electron/electron/issues/21445`
