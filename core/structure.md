# OpenClaw Structure Reference

File locations and configuration overview. For details see:
- `systemd.md` — how systemd works with Clawdbot
- `openclaw-auth.md` — authentication and provider setup
- `openclaw-gateway.md` — gateway service and Control UI
- `openclaw-channels.md` — Telegram and multi-agent setup
- `openclaw-memory.md` — memory system and persistence
- `openclaw-skills.md` — skills system and configuration

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
├── memory/
│   └── YYYY-MM-DD.md          # Daily logs (auto-appended)
├── .agents/skills/            # Installed skills (from npx skills add)
│   └── firecrawl/             # Example: Firecrawl skill
└── .firecrawl/                # Firecrawl output (gitignored)

~/.config/systemd/user/        # Systemd user services
├── clawdbot-gateway.service              # Main service (managed by Clawdbot)
└── clawdbot-gateway.service.d/           # Drop-ins (your customizations)
    └── *.conf                            # Skill credentials, env vars
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

## Security

- Keep gateway loopback-only — access via SSH tunnel
- Use pairing policy — unknown DMs require approval
- Enable sandbox mode for group chats
- Don't expose port 18789 unless using Tailscale
