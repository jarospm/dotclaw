# OpenClaw Skills

Skills extend what the agent can do. They're documentation + metadata that teach the agent how to use external tools.

---

## What Skills Are

A skill is:
1. A **SKILL.md file** with instructions for the agent
2. **Metadata** about requirements (which CLI binaries are needed)
3. **Install instructions** for the underlying tools

Skills don't contain the actual tools — they document how to use them.

Example: The `bird` skill tells the agent how to use the `bird` CLI for X/Twitter. But you still need to install `bird` separately.

---

## Bundled vs Community Skills

**Bundled:** Ship with Clawdbot
```
~/.npm-global/lib/node_modules/clawdbot/skills/
```

**Community:** Install from ClawdHub
```bash
npx clawdhub search <term>
npx clawdhub install <skill>
```

Browse: https://clawdhub.com

---

## List Skills

```bash
# List all skills and their status
clawdbot skills list

# Check a specific skill
clawdbot skills info <skill-name>
```

Status meanings:
- **✓ Ready** — The required CLI is installed
- **✗ Missing** — The skill exists but the CLI isn't installed

---

## Installing a Skill's CLI

When `clawdbot skills info <skill>` shows "Missing requirements", install the CLI:

```bash
# Example: bird (X/Twitter)
npm install -g @steipete/bird

# Example: gh (GitHub)
# Usually pre-installed or: apt install gh
```

After installing, `clawdbot skills list` will show ✓ Ready.

---

## Skills That Need Credentials

Some skills require API keys or tokens. These are set via environment variables on the gateway.

### Method: Systemd Drop-in

See `systemd.md` for full details.

```bash
# Create drop-in
mkdir -p ~/.config/systemd/user/clawdbot-gateway.service.d/
cat > ~/.config/systemd/user/clawdbot-gateway.service.d/<skill>.conf << 'EOF'
[Service]
Environment="API_KEY=your_key"
EOF

# Apply
systemctl --user daemon-reload
systemctl --user restart clawdbot-gateway
```

### Common Skills and Their Env Vars

| Skill | Env Vars | How to Get |
|-------|----------|------------|
| bird | `AUTH_TOKEN`, `CT0` | Extract from browser cookies on x.com |
| gog (Google) | `GOG_CLIENT_ID`, `GOG_CLIENT_SECRET` | Google Cloud Console |
| himalaya (email) | Configured via `~/.config/himalaya/config.toml` | N/A |

---

## How the Agent Uses Skills

1. Agent receives a request (e.g., "read this tweet")
2. Agent checks available skills
3. Finds `bird` skill, reads SKILL.md for instructions
4. Runs `bird read <url>` via exec
5. Returns result to user

The skill file teaches the agent *what commands to run*. The systemd env vars provide *credentials* for those commands.

---

## Creating Custom Skills

See the `skill-creator` skill:
```bash
clawdbot skills info skill-creator
```

Basic structure:
```
my-skill/
└── SKILL.md      # Instructions + metadata
```

SKILL.md format:
```markdown
---
name: my-skill
description: What this skill does
metadata: {"clawdbot":{"requires":{"bins":["my-cli"]}}}
---

# My Skill

Instructions for using my-cli...
```

---

## File Locations

```
# Bundled skills
~/.npm-global/lib/node_modules/clawdbot/skills/

# Skill credentials (systemd drop-ins)
~/.config/systemd/user/clawdbot-gateway.service.d/
```
