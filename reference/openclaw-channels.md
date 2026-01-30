# OpenClaw Channels

Configuring Telegram and other messaging channels.

---

## Telegram Setup

In `~/.clawdbot/clawdbot.json`:

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "botToken": "<BOT_TOKEN>",
      "dmPolicy": "pairing",
      "groupPolicy": "allowlist"
    }
  }
}
```

---

## DM Pairing

When `dmPolicy: "pairing"` is enabled, unknown users must be approved:

```bash
clawdbot pairing approve telegram <CODE>
clawdbot pairing list
```

Approved users stored in `~/.clawdbot/credentials/telegram-pairing.json`.

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

## File Locations

```
~/.clawdbot/credentials/telegram-pairing.json   # User approvals
~/.clawdbot/credentials/telegram-allowFrom.json # Allowlist
~/.clawdbot/telegram/                           # Session data
```
