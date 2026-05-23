---
name: linux-chief
description: Troubleshoot and enhance KDE Plasma 6 and Tuxedo OS Linux systems across audio, desktop, Docker, filesystems, video, AI tools, and performance.
---

# Linux Chief - Master Troubleshooting Skill

Central command for Linux troubleshooting and enhancements on KDE Plasma 6 / Tuxedo OS 24.04.

**System:** Tuxedo OS 24.04 (Ubuntu-based), KDE Plasma 6, X11, NVIDIA RTX 4070 Ti, PipeWire

## When to Use This Skill

Invoke when user mentions:
- Audio problems: "no sound", "audio crackling", "PipeWire", "Discord audio"
- Desktop issues: "Plasma crashed", "panel broken", "KDE", "theme"
- Docker/containers: "container", "port conflict", "docker", "WAHA", "WhatsApp"
- File management: "Dolphin", "NTFS", "can't write", "mount"
- Video: "DaVinci Resolve", "OBS", "video playback"
- AI tools: "ComfyUI", "ControlNet crash"
- Performance: "system slow", "high memory", "earlyoom", "swap full", "orphaned processes"
- General: "linux help", "fix my...", "how do I..."

## Quick Start - Troubleshooting Flow

### Step 1: Run Diagnostics
For general issues, run the system diagnostic:
```bash
bash .claude/skills/linux-chief/scripts/system-diagnostic.sh
```

For audio-specific issues:
```bash
bash .claude/skills/linux-chief/scripts/audio-diagnostic.sh
```

### Step 2: Check Quick Fixes
For immediate one-liner solutions, see [QUICK-FIXES.md](QUICK-FIXES.md)

### Step 3: Check Known Issues
For current bugs and workarounds, see [KNOWN-ISSUES.md](KNOWN-ISSUES.md)

### Step 4: Consult Documentation
Find the relevant guide in `docs/` using the index below.

---

## Documentation Index

### Audio & PipeWire
| Issue | Guide |
|-------|-------|
| Discord audio crackling | `docs/discord-audio-troubleshooting.md` |
| Flatpak apps no audio | `docs/flatpak-browser-audio.md` |
| "Too many connections" | `docs/pipewire-connection-exhaustion.md` |
| OBS audio/video issues | `docs/obs-virtual-camera-setup.md` |

### Video Production
| Issue | Guide |
|-------|-------|
| Installing DaVinci Resolve | `docs/davinci-resolve-install.md` |
| Converting files for Resolve | `docs/dolphin-convert-to-resolve.md` |

### File Management
| Issue | Guide |
|-------|-------|
| NTFS read/write access | `docs/ntfs-read-write-fix.md` |
| Downloads folder organization | `docs/downloads-symlink-setup.md` |

### Docker
| Issue | Guide |
|-------|-------|
| Docker setup | `docs/docker-linux/README.md` |
| Container URLs (dps) | `docs/docker-linux/dps-command.md` |
| WAHA WhatsApp API | `docs/waha-whatsapp-api-troubleshooting.md` |

### AI & ML
| Issue | Guide |
|-------|-------|
| ComfyUI setup | `docs/comfyui/README.md` |

### System
| Issue | Guide |
|-------|-------|
| GRUB/boot repair | `docs/grub-bootloader-repair.md` |
| Memory protection | `docs/memory-protection-setup.md` |
| Zellij terminal | `docs/zellij-autostart-layout.md` |

### Performance & Memory
| Issue | Guide |
|-------|-------|
| Memory protection (swap + earlyoom) | `docs/memory-protection-setup.md` |
| Orphaned processes cleanup | `/process-cleanup` skill |

---

## Route to Sub-Skills

For complex multi-step tasks, route to specialized skills:

| Request | Skill |
|---------|-------|
| "Make KDE look like macOS" | `/kde-macos-theme` |
| "Set up VS Code themes per project" | `/vscode-theme-setup` |
| "WAHA", "WhatsApp bot", "send WhatsApp message" | `/waha` |
| "Sound stopped", "no audio", "fix audio" | `/fix-audio` |
| "System slow", "memory high", "orphaned processes", "cleanup" | `/process-cleanup` |

---

## Troubleshooting Methodology

Follow this systematic approach:

1. **Define the problem** - What exactly is happening? What error messages?
2. **Gather information** - Run diagnostic scripts, check logs
3. **Check known issues** - Is this a documented bug with a workaround?
4. **Form hypothesis** - Based on symptoms, what's the likely cause?
5. **Test solution** - Apply fix, verify it works
6. **Document** - If new issue, offer to create doc in `docs/`

### Key Log Locations
```bash
# System logs
journalctl -xe                    # Recent with context
journalctl -p err -b              # Errors since boot
journalctl --user -u pipewire     # PipeWire logs

# Application logs
~/.local/share/kwin/scripts/      # KWin scripts
~/.xsession-errors                # X11 session errors
```

---

## Critical Safety Rules for Destructive Operations

**BEFORE any migration, deletion, or major system change:**

1. **Verify backups exist** - Ask user to confirm backup location
2. **Use dry-run first** - `rsync --dry-run`, `rm -i`, etc.
3. **Verify each step succeeded** - Check return codes, use `mountpoint -q`
4. **Move to trash, don't delete** - Use `mv` to backup location first
5. **Stop on ANY error** - Never continue after a failure

**NEVER run `rm -rf` on directories that could be mount points without first:**
```bash
# Verify NOT a mount point
! mountpoint -q /path/to/dir && rm -rf /path/to/dir
```

---

*Last updated: 2026-01-24*
