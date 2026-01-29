# Moltbot Structure Reference

File locations and configuration overview. For details see:
- `moltbot-auth.md` — authentication and provider setup
- `moltbot-gateway.md` — gateway service and Control UI
- `moltbot-channels.md` — Telegram and multi-agent setup

---

## File Locations

```
~/.clawdbot/
├── clawdbot.json              # Main config (gateway, agents, channels)
├── agents/
│   └── main/
│       ├── agent/
│       │   └── auth-profiles.json   # Provider credentials
│       └── sessions/
│           └── sessions.json        # Session store
├── credentials/               # Channel credentials
├── telegram/                  # Telegram session data
└── devices/                   # Device registrations

~/clawd/                       # Agent workspace
├── MEMORY.md                  # Long-term memory (curated)
├── IDENTITY.md                # Bot persona
├── USER.md                    # User profile
├── SOUL.md                    # Core values/behavior
└── memory/
    └── YYYY-MM-DD.md          # Daily logs (auto-appended)
```

---

## Configuration

Main config: `~/.clawdbot/clawdbot.json`

**Key settings:**
- `gateway.port` — default 18789
- `agents.main.model` — e.g., `anthropic/claude-opus-4-5`
- `agents.defaults.workspace` — agent working directory
- `agents.defaults.sandbox.mode` — `"non-main"` for Docker isolation

---

## Memory

Automatic persistence:

- **Daily logs** (`memory/YYYY-MM-DD.md`) — loaded at session start
- **Long-term** (`MEMORY.md`) — persists forever, curated
- **Auto-flush** — writes before context compaction

---

## Security

- Keep gateway loopback-only — access via SSH tunnel
- Use pairing policy — unknown DMs require approval
- Enable sandbox mode for group chats
- Don't expose port 18789 unless using Tailscale
