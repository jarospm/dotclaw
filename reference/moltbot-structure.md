# Moltbot Structure Reference

File locations, configuration, and day-to-day operations for an existing Moltbot installation.

---

## File Locations

```
~/.clawdbot/
├── clawdbot.json              # Main config (gateway, agents, bindings)
├── agents/
│   └── main/
│       ├── agent/
│       │   └── auth-profiles.json   # Provider credentials
│       └── sessions/
│           └── sessions.json        # Session store
├── credentials/               # Channel credentials
│   └── telegram-pairing.json  # Telegram user approvals
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
- `gateway.auth.token` — Control UI authentication
- `gateway.port` — Default 18789
- `agents.main.model` — e.g., `anthropic/claude-opus-4-5`
- `agents.defaults.sandbox.mode` — `"non-main"` for Docker isolation

---

## Authentication

### Profile types

- **setup-token** (type: "token") — 1 year validity, preferred for 24/7
- **OAuth** (type: "oauth") — 3 hour expiry, auto-refreshes

Clawdbot auto-imports Claude Code OAuth if present. Use setup-token for stability.

### Check status

```bash
cat ~/.clawdbot/agents/main/agent/auth-profiles.json
clawdbot models status
clawdbot doctor
```

### Refresh token

```bash
claude setup-token
clawdbot models auth paste-token --provider anthropic
```

### Set default profile

```bash
clawdbot models auth order get --provider anthropic
clawdbot models auth order set --provider anthropic anthropic:manual
```

Active profile stored in `auth-profiles.json` under `lastGood`.

---

## Gateway

Service: `clawdbot-gateway` (see `server-commands.md` for systemd)

```bash
clawdbot doctor              # Health check
clawdbot dashboard --no-open # Get Control UI link with token
```

---

## Memory

Automatic persistence:

- **Daily logs** (`memory/YYYY-MM-DD.md`) — loaded at session start
- **Long-term** (`MEMORY.md`) — persists forever, curated
- **Auto-flush** — writes before context compaction

---

## Telegram Pairing

If DM pairing policy is enabled:

```bash
clawdbot pairing approve telegram <CODE>
clawdbot pairing list
```

---

## Multi-Agent Setup

Multiple bots with different personas via one gateway:

```json5
{
  channels: {
    telegram: {
      accounts: {
        botone: { token: "TOKEN_1" },
        bottwo: { token: "TOKEN_2" }
      }
    }
  },
  agents: {
    main: { /* ... */ },
    secondary: {
      model: "anthropic/claude-sonnet-4",
      workspace: "/home/deploy/clawd/secondary"
    }
  },
  bindings: [
    { agentId: "main", match: { channel: "telegram", accountId: "botone" } },
    { agentId: "secondary", match: { channel: "telegram", accountId: "bottwo" } }
  ]
}
```

Restart after changes: `systemctl --user restart clawdbot-gateway`

---

## Security

- Keep Gateway loopback-only — access via SSH tunnel
- Use pairing policy — unknown DMs require approval
- Enable sandbox mode for group chats
- Don't expose port 18789 unless using Tailscale
