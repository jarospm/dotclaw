# OpenClaw Setup Guide

First-time installation and configuration of OpenClaw on a VPS.

---

## Prerequisites

- Ubuntu 22.04+ LTS
- Node.js ≥24 installed
- 4GB+ RAM (2GB for chat-only, 4GB+ for browser automation)
- SSH access as non-root user

See `../guides/server-setup.md` for base server configuration.

---

## Step 1: Install Clawdbot

> **Note:** The npm package `openclaw` is a squatter. The real package is `clawdbot`.

```bash
npm install -g clawdbot@latest
clawdbot --version
```

---

## Step 2: Choose AI Provider

### Option A: Setup-Token (Recommended)

Use your Claude Pro/Max subscription. Preferred for 24/7 bots — token lasts 1 year.

```bash
claude setup-token
clawdbot models auth paste-token --provider anthropic
```

### Option B: Anthropic API Key

Pay-per-use billing.

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
```

### Option C: Vercel AI Gateway

Pay-per-use via Vercel (does NOT use your Claude subscription).

Select during onboarding wizard.

---

## Step 3: Run Onboarding

```bash
clawdbot onboard --install-daemon
```

The `--install-daemon` flag creates a systemd service for 24/7 operation.

### Onboarding options

**Gateway bind:**
- Loopback (127.0.0.1) — secure, access via SSH tunnel (recommended)
- LAN (0.0.0.0) — public, needs firewall
- Tailnet — Tailscale only

**Gateway auth:**
- Token (recommended) — auto-generated
- Password — less secure
- Off — only safe with loopback

**Hatching:**
- TUI — interactive personality setup in terminal
- Web UI — browser-based
- Later — configure via MEMORY.md manually

---

## Step 4: Configure Channels

### Telegram

1. Create bot via [@BotFather](https://t.me/botfather)
2. Copy token
3. Enter during onboarding

### WhatsApp

1. Scan QR code during onboarding
2. Phone stays linked

### Discord

1. Create app at [discord.com/developers](https://discord.com/developers/applications)
2. Copy bot token
3. Enter during onboarding

---

## Step 5: Verify

```bash
clawdbot doctor
systemctl --user status clawdbot-gateway
```

Access Control UI via SSH tunnel:
```bash
ssh -N -L 18789:127.0.0.1:18789 <your-server>
```
Then open http://127.0.0.1:18789/

---

## Troubleshooting

**Node version too old:** `node -v` must be ≥24

**Can't connect to Gateway:**
```bash
systemctl --user status clawdbot-gateway
journalctl --user -u clawdbot-gateway -f
```

**Auth token expired:**
```bash
claude setup-token
clawdbot models auth paste-token --provider anthropic
```

---

## Resources

- [Official Docs](https://docs.openclaw.dev/)
- [GitHub](https://github.com/openclaw/openclaw)
- [Discord Community](https://discord.gg/openclaw)
