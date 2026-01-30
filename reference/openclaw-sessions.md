# OpenClaw Sessions

A session is a conversation transcript with history — like a chat thread that persists.

---

## Session Types

- **Main session** (`agent:main:main`) — All your DMs route here by default
- **Group sessions** (`agent:main:<channel>:group:<id>`) — Each group gets its own
- **Cron sessions** (`cron:<jobId>`) — Isolated jobs get fresh sessions per run

---

## Key Concept: DMs Share One Session

By default, all your direct messages — from Telegram, WhatsApp, CLI, etc. — route to **one main session**. This gives you continuity across devices and channels.

Group chats are isolated — each group gets its own session.

---

## Inspecting Sessions

```bash
clawdbot sessions              # list all sessions with token usage
clawdbot sessions --active 60  # only sessions active in last hour
clawdbot sessions --json       # machine-readable output
```

In chat:
- `/status` — quick context usage check
- `/context list` — what's injected into the system prompt
- `/context detail` — deeper breakdown (tool schemas, skills, files)

---

## Context Accumulation

Sessions accumulate context with every message, tool call, and result.

Check usage:
```bash
clawdbot sessions
# Shows: agent:main:main  154k/200k (77%)
```

---

## Compaction (What Happens at Limits)

When a session nears the model's context window:

1. **Auto-compaction** — summarizes older messages, keeps recent ones
2. **Memory flush** — agent can write important notes to disk before compacting
3. **Summary persists** — stored in session JSONL, used for future requests

Manual compaction:
```
/compact                           # summarize older history
/compact Focus on decisions        # with instructions
```

**Important:** `/compact` is a text-only command — it won't show in Telegram's `/` menu, but you can type it manually and it works.

---

## Session Resets

Default: **daily reset at 4:00 AM** (gateway local time).

Manual reset:
```
/new              # start fresh session
/reset            # same as /new
/new opus         # fresh session with specific model
```

`/new` and `/reset` are identical — both start a fresh session.

### Memory Is NOT Saved on Manual Reset

**Critical:** The automatic memory flush only triggers before **auto-compaction** (when near full). Manual `/new` or `/reset` does NOT save memories — unsaved context is lost.

**Best practice for manual reset:**

1. Ask the agent to save first:
   ```
   Save anything important to memory files, then I'll reset
   ```

2. Then reset:
   ```
   /new
   ```

### Compaction vs Reset

| Action | Context | Memory saved? | Use when |
|--------|---------|---------------|----------|
| `/compact` | Summarized, keeps recent | No auto-save | Free up space, keep continuity |
| `/new` | Wiped completely | No auto-save | Fresh start, new topic |
| Auto-compaction | Summarized automatically | Yes (memory flush) | Happens at ~90% full |

**Prefer `/compact` over `/new`** when you just want to free up space — you keep continuity and can add instructions about what to preserve.

---

## Heartbeat (Periodic Wake-ups)

Heartbeat is a periodic wake-up that runs in the **main session**. It's how the agent checks on things proactively without you messaging first.

### How It Works

1. Gateway triggers heartbeat at configured interval (default: 30 min)
2. Agent reads `HEARTBEAT.md` in its workspace for instructions
3. If `HEARTBEAT.md` is empty — agent replies `HEARTBEAT_OK` and skips
4. If tasks are listed — agent executes them with full main session context
5. Output can be delivered to a channel or stay internal

### The HEARTBEAT.md File

Located in the agent's workspace (e.g., `~/clawd/HEARTBEAT.md`).

```markdown
# Heartbeat checklist

- Check email for urgent messages
- Review calendar for events in next 2 hours
- If idle for 8+ hours, send a brief check-in
```

Keep it small — the file is read every heartbeat, so it burns tokens.

### Connection to Sessions

- Heartbeat **always runs in the main session** — it has full conversation context
- System events queued by cron (`sessionTarget: "main"`) get processed during heartbeat
- If `HEARTBEAT.md` is empty, system events sit unprocessed until you message

This is why the gym reminder failed:
1. Cron queued a system event into main session
2. `wakeMode: "now"` triggered a heartbeat
3. Heartbeat checked `HEARTBEAT.md` — it was empty
4. Heartbeat skipped — event never got processed

### Configuration

In `~/.clawdbot/clawdbot.json`:

```json
{
  "agents": {
    "defaults": {
      "heartbeat": {
        "every": "30m",
        "target": "last",
        "activeHours": { "start": "08:00", "end": "22:00" }
      }
    }
  }
}
```

- `every` — interval between heartbeats
- `target` — where to deliver output (`last` = last channel you messaged from)
- `activeHours` — optional quiet hours (no heartbeats outside this window)

### Heartbeat vs Cron

| | Heartbeat | Cron |
|---|---|---|
| Session | Main (shared context) | Main or isolated |
| Timing | Interval-based, can drift | Exact (cron expressions) |
| Instructions | `HEARTBEAT.md` | Job payload |
| Best for | Batched checks, context-aware tasks | Precise schedules, push notifications |

**Use heartbeat when:**
- Multiple checks can batch together (inbox + calendar + notifications)
- Task benefits from conversation context
- Timing can drift slightly

**Use cron when:**
- Exact timing matters ("9:00 AM sharp")
- You need push notifications (isolated + deliver)
- Task should run independently of main session state

---

## Cron Jobs (Scheduled Tasks)

Cron is the built-in scheduler. Jobs persist in `~/.clawdbot/cron/jobs.json`.

### Two Execution Modes

**Main session** (`sessionTarget: "main"`)
- Queues a system event into the main session
- Processed on next heartbeat — **not** a push notification
- Use for context-aware reminders the agent handles internally

**Isolated session** (`sessionTarget: "isolated"`)
- Spins up a fresh `cron:<jobId>` session
- Runs an agent turn and can deliver output to a channel
- **This is how you get push notifications**

### For Push Notifications

```json
{
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "message": "Your reminder text",
    "deliver": true,
    "channel": "telegram",
    "to": "<chat-id>"
  }
}
```

### Cron CLI Commands

```bash
clawdbot cron list                    # list all jobs
clawdbot cron status                  # jobs with next run times
clawdbot cron add --help              # see add options
clawdbot cron remove <jobId>          # delete a job
clawdbot cron run <jobId> --force     # manually trigger
clawdbot cron runs --id <jobId>       # view run history
```

### Example: One-shot Reminder (Push to Telegram)

```bash
clawdbot cron add \
  --name "Call reminder" \
  --at "20m" \
  --session isolated \
  --message "Time to make that call" \
  --deliver \
  --channel telegram \
  --to "<chat-id>" \
  --delete-after-run
```

### Example: Recurring Job

```bash
clawdbot cron add \
  --name "Morning check" \
  --cron "0 8 * * *" \
  --tz "Europe/Stockholm" \
  --session isolated \
  --message "Morning briefing: calendar, weather, emails" \
  --deliver \
  --channel telegram \
  --to "<chat-id>"
```

### Cleanup

Jobs with `deleteAfterRun: true` only delete after **successful** runs.
Skipped/failed jobs stay in the store — remove manually with `clawdbot cron remove`.

---

## Session Storage

- **Store file:** `~/.clawdbot/agents/main/sessions/sessions.json`
- **Transcripts:** `~/.clawdbot/agents/main/sessions/<SessionId>.jsonl`

---

## Configuration

In `~/.clawdbot/clawdbot.json`:

```json
{
  "session": {
    "dmScope": "main",
    "reset": {
      "mode": "daily",
      "atHour": 4
    }
  }
}
```

Options for `dmScope`:
- `main` — all DMs share one session (default)
- `per-peer` — isolate by sender
- `per-channel-peer` — isolate by channel + sender

---

## Slash Commands (Telegram)

Commands come in two types:

**Native commands** — show in Telegram's `/` autocomplete menu:
- `/new`, `/reset`, `/status`, `/stop`, `/help`, `/model`, etc.

**Text-only commands** — must be typed manually (not in menu):
- `/compact` — summarize history
- `/context list` — show what's injected
- `/context detail` — detailed breakdown

Both types work — text-only commands just don't appear in autocomplete.

---

## Commands Summary

| Command | What it does |
|---------|--------------|
| `clawdbot sessions` | List sessions with token usage |
| `clawdbot cron list` | List scheduled jobs |
| `clawdbot cron remove <id>` | Delete a cron job |
| `clawdbot system event --text "..."` | Queue a system event |
| `/status` | Quick context check in chat |
| `/context list` | What's in the system prompt (text-only) |
| `/compact` | Manually summarize old history (text-only) |
| `/new` | Start fresh session |

---

## Tips

- **Context accumulates** — use `/compact` when sessions feel bloated (around 80%)
- **Prefer `/compact` over `/new`** — keeps continuity, just frees space
- **Ask agent to save before `/new`** — manual reset doesn't trigger memory flush
- **Important info belongs in files** — `memory/`, `MEMORY.md` survive everything
- **Check usage regularly** — `clawdbot sessions` or `/status` in chat
- **Isolated cron sessions are disposable** — they don't accumulate, ignore them
- **Daily reset at 4 AM** — sessions auto-reset overnight by default
