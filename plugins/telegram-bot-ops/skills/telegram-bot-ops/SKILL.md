---
name: telegram-bot-ops
description: Production operations guide for python-telegram-bot v20+ community bots, covering polling vs webhooks, forum topics, scheduling, rate limits, dashboard integration, and deployment.
---

# Telegram Bot Operations — python-telegram-bot v20+

## Quick Reference

| Area | Recommendation |
|---|---|
| Transport | Webhooks behind nginx for production; polling only for dev |
| Duplicate prevention | PID lock file for polling; webhook URL is a natural lock |
| Forum topics | Pyrogram seed at startup + PTB event tracking for updates |
| Admin permissions | Bot MUST be admin to see messages in forum groups |
| Scheduler | PTB JobQueue; re-register from config in `post_init` |
| Timezone | Use `zoneinfo.ZoneInfo` not pytz |
| Config reload | SIGHUP handler or internal aiohttp API |
| SQLite concurrency | Enable WAL mode + 5s busy timeout |
| Rate limits | `AIORateLimiter`, stagger multi-topic sends by 3s+ |

---

## 1. Polling vs Webhooks

**Polling** — simple, no public URL needed, fine for dev. Bot makes repeated requests.
**Webhooks** — production choice. Telegram pushes updates instantly. Lower latency, zero idle load.

### Webhook setup with PTB v20+

```python
app.run_webhook(
    listen="0.0.0.0",
    port=8443,
    secret_token="YourSecretTokenHere",
    webhook_url="https://your-domain.com:8443",
)
```

Behind nginx: let nginx handle SSL, forward to bot on local port. Supported ports: 443, 80, 88, 8443.

### Prevent duplicate instances

**PID lock file:**
```python
import os, sys, atexit

PID_FILE = "/tmp/mybot.pid"

def acquire_lock():
    if os.path.exists(PID_FILE):
        with open(PID_FILE) as f:
            old_pid = int(f.read().strip())
        try:
            os.kill(old_pid, 0)
            print(f"Bot already running (PID {old_pid}). Exiting.")
            sys.exit(1)
        except ProcessLookupError:
            pass
    with open(PID_FILE, "w") as f:
        f.write(str(os.getpid()))
    atexit.register(lambda: os.unlink(PID_FILE))
```

---

## 2. Forum Topics Discovery

Bot API has **NO `getForumTopics` method**. Workarounds:

### A. Event tracking (pure PTB)
Listen for `filters.StatusUpdate.FORUM_TOPIC_CREATED` and `FORUM_TOPIC_EDITED`. Track `message_thread_id` from all messages. Only discovers topics with activity after bot was added.

### B. Pyrogram hybrid (recommended for full topic list)
Pyrogram supports `get_forum_topics()` via MTProto. Run once at startup to seed DB:

```python
from pyrogram import Client

async with Client("seeder", bot_token=TOKEN, api_id=ID, api_hash=HASH) as client:
    topics = await client.get_forum_topics(chat_id)
    for t in topics:
        db.upsert_forum_topic(t.id, t.title)
```

Requires `api_id` + `api_hash` from my.telegram.org even for bot tokens.

### C. Message-based tracking (current approach)
Track `message_thread_id` from every incoming message. Name fallback: `"Topic {id}"` until a `forum_topic_created` event provides the real name.

**Known limitation:** General topic has `message_thread_id = None` — can't be tracked this way.

---

## 3. Admin Permissions

**Bots MUST be admin to see all messages in forum groups.** Privacy mode off is NOT enough — bot must be removed and re-added after changing the setting.

Making bot admin bypasses privacy mode automatically. Minimum permissions:

| Permission | Why |
|---|---|
| Read messages | See all messages in all topics |
| Delete messages | Anti-spam |
| Restrict members | Mute/ban |
| Manage topics | Create/edit/close topics |
| Pin messages | Optional |

---

## 4. Scheduler

### PTB JobQueue (recommended)
Wrapper around APScheduler. Jobs are in-memory — lost on restart. Re-register from config in `post_init`:

```python
async def post_init(app):
    settings = load_settings()
    schedule = settings["schedule"]
    morning = schedule["morning_prompt"]
    app.job_queue.run_daily(
        send_morning_prompt,
        time=parse_time(morning["time"]),
        days=tuple(morning["days"]),
    )
```

### Hot-reload without restart
```python
async def reload_schedule(app):
    for job in app.job_queue.jobs():
        job.schedule_removal()
    register_all_jobs(app)  # re-read config
```

### Timezone
```python
from zoneinfo import ZoneInfo
from datetime import time
tz = ZoneInfo("Asia/Jerusalem")
job_time = time(9, 0, tzinfo=tz)
```

---

## 5. Dashboard ↔ Bot Communication

### Option A: Shared SQLite + polling (simplest)
Dashboard writes config. Bot polls DB every 30s for changes via `job_queue.run_repeating`.

### Option B: SIGHUP signal
Dashboard sends `os.kill(pid, signal.SIGHUP)`. Bot catches it and reloads config.

### Option C: Internal HTTP API (most flexible)
Run aiohttp server inside the bot. Dashboard POSTs actions to `http://localhost:PORT/internal/trigger`.

### SQLite concurrent access
Enable WAL mode on ALL connections:
```python
conn.execute("PRAGMA journal_mode=WAL")
conn.execute("PRAGMA busy_timeout=5000")
```

---

## 6. Rate Limits

| Scope | Limit |
|---|---|
| Single chat | ~1 msg/second |
| Group messages | 20 msgs/minute |
| Global broadcast | ~30 msgs/second |

### Built-in rate limiter
```python
from telegram.ext import AIORateLimiter
app = ApplicationBuilder().token(TOKEN).rate_limiter(AIORateLimiter(max_retries=3)).build()
```

### Handle 429 errors
```python
from telegram.error import RetryAfter
try:
    await bot.send_message(...)
except RetryAfter as e:
    await asyncio.sleep(e.retry_after + 1)
    await bot.send_message(...)  # retry
```

### Multi-topic staggering
```python
for i, topic in enumerate(topics):
    await asyncio.sleep(i * 3)  # 3s between topics
    await bot.send_message(chat_id=GROUP_ID, text=msg, message_thread_id=topic.id)
```

---

## 7. Production Deployment

### Dockerfile
```dockerfile
FROM python:3.12-slim
RUN useradd --create-home botuser
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY --chown=botuser:botuser . .
USER botuser
CMD ["python", "-u", "bot.py"]  # exec form for signal handling
```

### Docker Compose
```yaml
services:
  bot:
    build: .
    restart: unless-stopped
    stop_grace_period: 30s
    logging:
      driver: json-file
      options:
        max-size: "20m"
        max-file: "5"
```

### Graceful shutdown
PTB v20+ handles SIGTERM/SIGINT automatically with `run_polling()` / `run_webhook()`. Use `post_shutdown` hook for cleanup.

---

## Known Gaps in This Project (robotnik)

1. **No PID lock** — multiple bot instances can run simultaneously, causing update conflicts. Add T-052.
2. **No hot-reload** — changing dashboard config requires bot restart. Add SIGHUP handler.
3. **No Pyrogram topic seed** — only tracks topics from messages. Add one-time Pyrogram seed.
4. **No WAL mode** — SQLite concurrent access from dashboard + bot may cause SQLITE_BUSY.
5. **No rate limiter** — no `AIORateLimiter` configured. Add to Application builder.
6. **Polling in production** — should switch to webhooks when deploying to VPS.
7. **General topic (thread_id=None)** — can't be tracked or targeted by thread_id.
