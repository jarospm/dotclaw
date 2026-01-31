# OpenClaw Authentication

How OpenClaw authenticates with AI providers (Anthropic).

---

## Profile Types

- **setup-token** (type: `"token"`) — 1 year validity, preferred for 24/7 servers
- **OAuth** (type: `"oauth"`) — 3 hour expiry, auto-refreshes

Clawdbot auto-imports Claude Code OAuth if present. Use setup-token for stability.

---

## Check Status

```bash
cat ~/.clawdbot/agents/main/agent/auth-profiles.json
clawdbot models status
clawdbot doctor
```

The `auth-profiles.json` file shows:
- `profiles` — all configured auth methods
- `lastGood` — which profile is actively being used
- `usageStats` — last used timestamp and error count

---

## Create Setup Token

```bash
claude setup-token
clawdbot models auth paste-token --provider anthropic
```

---

## Set Default Profile

```bash
clawdbot models auth order get --provider anthropic
clawdbot models auth order set --provider anthropic anthropic:manual
```

Active profile stored in `auth-profiles.json` under `lastGood`.

---

## File Location

```
~/.clawdbot/agents/main/agent/auth-profiles.json
```
