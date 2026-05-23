---
name: fix-audio
description: Diagnose and fix PipeWire audio failures caused by connection exhaustion, zombie clients, and sudden no-sound issues on Linux.
---

# Fix Audio - PipeWire Connection Exhaustion Fix

Quick fix for when audio suddenly stops working due to PipeWire connection exhaustion.

## When to Use

Invoke with `/fix-audio` when:
- Audio suddenly stops working
- "No sound" in apps like Discord, Telegram, Ferdium
- Apps that were working fine suddenly have no audio
- PipeWire shows "too many connections" errors

## Step 1: Diagnose

Run diagnostics to confirm the issue:

```bash
# Count zombie connections (>50 is problematic, limit is ~64)
wpctl status | grep -c libcanberra

# Check for connection errors
systemctl --user status pipewire-pulse | grep -i "too many"
```

## Step 2: Restart PipeWire Services

```bash
systemctl --user restart pipewire pipewire-pulse wireplumber pipewire-pulse.socket
```

## Step 3: Verify PipeWire is Healthy

```bash
# Should show 0 or very low number
wpctl status | grep -c libcanberra

# Should connect successfully
pactl info | head -3
```

## Step 4: Restart Affected Apps

**IMPORTANT:** Apps cache the failed connection and MUST be restarted to get audio back.

### App Restart Checklist

Go through each app and restart it:

| App | Kill Command | Launch Command | Status |
|-----|--------------|----------------|--------|
| **Ferdium** | `pkill -9 -f ferdium` | `flatpak run org.ferdium.Ferdium &` | [ ] |
| **Telegram** | `pkill -9 -f telegram` | `flatpak run org.telegram.desktop &` | [ ] |
| **Vesktop (Discord)** | `pkill -9 -f vesktop` | `flatpak run dev.vencord.Vesktop &` | [ ] |
| **Zen Browser** | `pkill -9 -f zen` | `flatpak run app.zen_browser.zen &` | [ ] |
| **Zoom** | `pkill -9 -f zoom` | `zoom &` | [ ] |
| **Spotify** | `pkill -9 -f spotify` | `flatpak run com.spotify.Client &` | [ ] |

### Quick Restart All Common Apps

```bash
# Kill all
pkill -9 -f ferdium
pkill -9 -f telegram
pkill -9 -f vesktop
sleep 2

# Relaunch
flatpak run org.ferdium.Ferdium &>/dev/null &
flatpak run org.telegram.desktop &>/dev/null &
flatpak run dev.vencord.Vesktop &>/dev/null &
```

## Step 5: Test Audio

1. Play a notification sound in each app
2. Test microphone in Discord/Telegram voice settings
3. Play media in browser

## Root Cause

PipeWire has a connection limit (~64 default). Over long sessions, **libcanberra** (KDE notification sounds) leaks connections that don't get cleaned up. After days of uptime, you hit the limit and new apps can't connect.

## Prevention

The system has these preventive measures:
- `~/.config/pipewire/pipewire-pulse.conf.d/99-max-clients.conf` - Increases limit to 1024
- `~/.config/systemd/user/pipewire-refresh.timer` - Health check every 2 hours

If this keeps happening, check the timer is running:
```bash
systemctl --user status pipewire-refresh.timer
```

## Full Documentation

See `docs/pipewire-connection-exhaustion.md` for detailed history and technical details.

---

*Last updated: 2026-01-18*
