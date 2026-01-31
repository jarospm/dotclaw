# Cron

Built-in scheduler for time-based jobs. Jobs persist in `~/.clawdbot/cron/jobs.json`.

## Two Execution Modes

### Main Session (`sessionTarget: "main"`)

- Queues a system event into the main session
- Processed on next heartbeat — **not** a push notification
- Use for context-aware reminders the agent handles internally

```json
{
  "sessionTarget": "main",
  "wakeMode": "next-heartbeat",
  "payload": {
    "kind": "systemEvent",
    "text": "Time to check on the project"
  }
}
```

⚠️ **Requires non-empty HEARTBEAT.md** — otherwise the event sits unprocessed.

### Isolated Session (`sessionTarget: "isolated"`)

- Spins up a fresh `cron:<jobId>` session
- Runs an agent turn and can deliver output to a channel
- **This is how you get push notifications**

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

## Push Notification Pattern

For reminders that need to reach you immediately:

```json
{
  "name": "standup-reminder",
  "schedule": { "kind": "cron", "expr": "0 9 * * 1-5" },
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "message": "Standup in 15 minutes!",
    "deliver": true,
    "channel": "telegram",
    "to": "58295755"
  }
}
```

## wakeMode (for isolated sessions)

Isolated jobs **always post a summary to main session** after completion. The `wakeMode` controls what happens next:

| wakeMode | Summary posted | Main session |
|----------|----------------|--------------|
| `next-heartbeat` (default) | Immediately | Waits for next scheduled heartbeat to see it |
| `now` | Immediately | Immediately triggered to run (sees the summary right away) |

**What this means:**
- The summary is always posted immediately to main — `wakeMode` doesn't change that
- `wakeMode` controls whether main session **wakes up to react** to the summary
- For push notifications with `deliver: true`, `wakeMode` usually doesn't matter — output goes directly to Telegram/etc, and the summary in main is just a log entry

**When `wakeMode: "now"` matters:**
- You want the main session to chain actions based on the cron result
- Example: Cron fetches data → posts summary → main session immediately analyzes it

**When `wakeMode: "next-heartbeat"` is fine:**
- Cron delivers output directly to a channel
- Summary in main is just for your records
- No follow-up action needed

## CLI Commands

```bash
clawdbot cron list                    # list all jobs
clawdbot cron status                  # jobs with next run times
clawdbot cron add --help              # see add options
clawdbot cron remove <jobId>          # delete a job
clawdbot cron run <jobId> --force     # manually trigger
clawdbot cron runs --id <jobId>       # view run history
```

## Example: One-shot Reminder

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

## Example: Recurring Job

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

## Agent Tool Access

The agent can manage cron via the `cron` tool:

```json
{
  "action": "add",
  "job": {
    "name": "my-reminder",
    "schedule": { "kind": "cron", "expr": "0 18 * * *" },
    "sessionTarget": "isolated",
    "payload": { ... }
  }
}
```

Actions: `status`, `list`, `add`, `update`, `remove`, `run`, `runs`

## Cleanup

- Jobs with `deleteAfterRun: true` auto-delete after **successful** runs
- Skipped/failed jobs stay — remove manually with `clawdbot cron remove`

## Jobs File

Direct access: `~/.clawdbot/cron/jobs.json`

Useful when:
- API times out
- Need to bulk edit jobs
- Debugging job state

## Quick Reference

| Want | Use |
|------|-----|
| Push notification | `sessionTarget: isolated` + `deliver: true` |
| Context-aware task | `sessionTarget: main` + `wakeMode: next-heartbeat` |
| One-shot reminder | Add `deleteAfterRun: true` |
| Exact time | Cron expression: `0 9 * * *` |
| Relative time | `--at "20m"` or `--at "2h"` |
