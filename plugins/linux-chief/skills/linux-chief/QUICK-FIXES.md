# Quick Fixes - One-Liner Solutions

Common problems with immediate solutions. For detailed troubleshooting, use `/linux-chief`.

## Audio

### No Sound After Suspend
```bash
systemctl --user restart pipewire pipewire-pulse wireplumber
```

### Apps Lost Audio (PipeWire connection exhaustion)
Use `/fix-audio` skill, then restart affected apps (Ferdium, Telegram, Vesktop, Zen Browser).

### Discord Audio Crackling
```bash
# Increase buffer size in Discord settings: Voice & Video > Audio Subsystem > Legacy
# Or restart PipeWire with larger quantum:
pw-metadata -n settings 0 clock.force-quantum 1024
```

## Performance

### System Slow / High Memory
Use `/process-cleanup` skill to find and kill orphaned processes.

Quick cleanup:
```bash
# Kill stuck Claude swarm agents
pkill -f "claude.*--max-turns" 2>/dev/null

# Kill orphaned vitest workers (can use 2-4GB each!)
pkill -f "vitest" 2>/dev/null

# Check memory
free -h
```

### Swap Full
```bash
# Check what's using swap
for f in /proc/*/status; do awk '/^(VmSwap|Name):/{printf $2 " "}/^VmSwap/ && $2>0{print ""}' "$f" 2>/dev/null; done | sort -k2 -n -r | head -10

# Clear swap (requires free RAM)
sudo swapoff -a && sudo swapon -a
```

## Docker

### Port Already in Use
```bash
# Find what's using port 3000
sudo lsof -i :3000

# Kill it
sudo kill $(sudo lsof -t -i :3000)
```

### Container Won't Start
```bash
# Check logs
docker logs <container_name> --tail 50

# Remove and recreate
docker rm -f <container_name>
docker compose up -d
```

## File System

### NTFS Drive Read-Only
```bash
# Remove Windows fast startup lock
sudo ntfsfix /dev/sdXN

# Or mount with write permissions
sudo mount -o rw,uid=$(id -u),gid=$(id -g) /dev/sdXN /mount/point
```

### Dolphin Not Showing Files
```bash
# Rebuild file index
baloo_file_extractor --rebuild
```

## KDE Plasma

### Panel Crashed / Missing
```bash
# Restart Plasma
kquitapp5 plasmashell && kstart5 plasmashell
```

### KWin Effects Broken
```bash
# Reset compositor
kwin_x11 --replace &
```

### Desktop Froze (Not System Freeze)
```bash
# Kill and restart KWin
pkill -9 kwin && kwin_x11 --replace &
```

## Network

### DNS Not Resolving
```bash
# Flush DNS cache
sudo systemd-resolve --flush-caches

# Check status
resolvectl status
```

### VPN Killing Network
```bash
# Reset network stack
sudo systemctl restart NetworkManager
```

---

*For complex issues, use `/linux-chief` for guided troubleshooting.*

*Last updated: 2026-01-24*
