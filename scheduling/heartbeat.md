# Heartbeat

Periodic wake-up that runs in the **main session**. How the agent checks on things proactively without you messaging first.

## How It Works

1. Gateway triggers heartbeat at configured interval (default: 30 min)
2. Agent reads `HEARTBEAT.md` in its workspace for instructions
3. If `HEARTBEAT.md` is empty — agent replies `HEARTBEAT_OK` and skips
4. If tasks are listed — agent executes them with full main session context
5. Output can be delivered to a channel or stay internal

## The HEARTBEAT.md File

Located in the agent's workspace (e.g., `~/clawd/HEARTBEAT.md`).

```markdown
# Heartbeat checklist

- Check email for urgent messages
- Review calendar for events in next 2 hours
- Check Moltbook for notable posts
```

Keep it small — the file is read every heartbeat, so it burns tokens.

## Connection to Sessions & Cron

- Heartbeat **always runs in the main session** — it has full conversation context
- System events queued by cron (`sessionTarget: "main"`) get processed during heartbeat
- If `HEARTBEAT.md` is empty, system events sit unprocessed until you message

**This is why heartbeat-style cron can fail:**
1. Cron queues a system event into main session
2. `wakeMode: "next-heartbeat"` waits for heartbeat
3. Heartbeat checks `HEARTBEAT.md` — it's empty
4. Heartbeat skips — event never gets processed

**Solution:** Either keep `HEARTBEAT.md` populated, or use isolated cron for push notifications.

## Configuration

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

## Heartbeat vs Cron

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

## State Tracking

Track check timestamps in `memory/heartbeat-state.json`:

```json
{
  "lastEmailCheck": "2026-01-30T14:00:00Z",
  "lastMoltbookCheck": "2026-01-30T14:00:00Z"
}
```

Prevents over-checking when heartbeats fire frequently.
