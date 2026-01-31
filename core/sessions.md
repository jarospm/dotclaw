# Sessions

A session is a conversation transcript with history — like a chat thread that persists.

## Session Types

| Type | Key Pattern | Description |
|------|-------------|-------------|
| Main | `agent:main:main` | All DMs route here by default |
| Group | `agent:main:<channel>:group:<id>` | Each group chat gets its own |
| Cron | `cron:<jobId>` | Isolated jobs get fresh sessions per run |

## Key Concept: DMs Share One Session

By default, all direct messages — from Telegram, WhatsApp, CLI, etc. — route to **one main session**. This gives you continuity across devices and channels.

Group chats are isolated — each group gets its own session.

## Inspecting Sessions

```bash
clawdbot sessions              # list all sessions with token usage
clawdbot sessions --active 60  # only sessions active in last hour
clawdbot sessions --json       # machine-readable output
```

In chat:
- `/status` — quick context usage check
- `/context list` — what's injected into the system prompt
- `/context detail` — deeper breakdown

## Context Accumulation

Sessions accumulate context with every message, tool call, and result.

```bash
clawdbot sessions
# Shows: agent:main:main  154k/200k (77%)
```

## Compaction

When a session nears the context window limit:

1. **Auto-compaction** — summarizes older messages, keeps recent ones
2. **Memory flush** — agent gets a silent turn to write important notes to disk
3. **Summary persists** — stored in session JSONL for future requests

Manual compaction:
```
/compact                           # summarize older history
/compact Focus on decisions        # with instructions
```

## Session Resets

Default: **daily reset at 4:00 AM** (gateway local time).

Manual reset:
```
/new              # start fresh session
/reset            # same as /new
/new opus         # fresh session with specific model
```

### Memory Is NOT Saved on Manual Reset

The automatic memory flush only triggers before **auto-compaction**. Manual `/new` does NOT save memories.

**Best practice:**
```
Save anything important to memory files, then I'll reset
```
Then: `/new`

### Compaction vs Reset

| Action | Context | Memory saved? | Use when |
|--------|---------|---------------|----------|
| `/compact` | Summarized, keeps recent | No auto-save | Free up space, keep continuity |
| `/new` | Wiped completely | No auto-save | Fresh start, new topic |
| Auto-compaction | Summarized automatically | Yes (memory flush) | Happens at ~90% full |

**Prefer `/compact` over `/new`** when you just want to free up space.

## Session Storage

- **Store file:** `~/.clawdbot/agents/main/sessions/sessions.json`
- **Transcripts:** `~/.clawdbot/agents/main/sessions/<SessionId>.jsonl`

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

## Slash Commands

**Native commands** (show in Telegram's `/` menu):
- `/new`, `/reset`, `/status`, `/stop`, `/help`, `/model`

**Text-only commands** (type manually):
- `/compact` — summarize history
- `/context list` — show what's injected
- `/context detail` — detailed breakdown

## Tips

- **Context accumulates** — use `/compact` around 80% full
- **Prefer `/compact` over `/new`** — keeps continuity
- **Ask agent to save before `/new`** — manual reset doesn't flush memory
- **Important info belongs in files** — `memory/`, `MEMORY.md` survive everything
- **Daily reset at 4 AM** — sessions auto-reset overnight
