# OpenClaw Gateway

The gateway is the long-running service that connects channels (Telegram, etc.) to agents.

---

## Service

Runs as systemd user service: `clawdbot-gateway`

```bash
systemctl --user status clawdbot-gateway
systemctl --user restart clawdbot-gateway
systemctl --user stop clawdbot-gateway
```

See:
- `systemd.md` — How systemd works, drop-in files, adding skill credentials
- `openclaw-skills.md` — Skills system and configuration

---

## Commands

```bash
clawdbot doctor              # Health check
clawdbot dashboard --no-open # Get Control UI link with token
```

---

## Configuration

In `~/.clawdbot/clawdbot.json`:

```json
{
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "loopback",
    "auth": {
      "mode": "token",
      "token": "<your-token>"
    }
  }
}
```

---

## Security

- Keep gateway loopback-only — access via SSH tunnel
- Don't expose port 18789 unless using Tailscale
- Use `gateway.auth.token` for Control UI authentication
