# Bird — X/Twitter CLI

Fast CLI for reading X/Twitter content via GraphQL + cookie auth.

---

## Skill Type

**Bundled** — comes with Clawdbot, no separate skill install needed.

| Component | Location |
|-----------|----------|
| Skill (SKILL.md) | `~/.npm-global/lib/node_modules/clawdbot/skills/bird/SKILL.md` |
| CLI binary | `~/.npm-global/bin/bird` |
| Credentials | `~/.config/systemd/user/clawdbot-gateway.service.d/bird.conf` |

---

## What It Does

- Read tweets, threads, replies
- Fetch bookmarks and likes
- Search tweets
- Access list timelines
- Fetch user profiles and timelines

**Not recommended for:** Tweeting/replying (triggers anti-bot blocks)

---

## How It Works

Bird uses X's undocumented web GraphQL API with your browser cookies for authentication. It looks like you browsing X, not a bot.

```
Your X cookies (auth_token, ct0)
        ↓
    Bird CLI
        ↓
  X GraphQL API
        ↓
   Tweet data
```

---

## Installation

**Package:**
```bash
npm install -g @steipete/bird
```

**Verify:**
```bash
which bird
# /home/deploy/.npm-global/bin/bird
```

---

## Skill Location

Bundled with Clawdbot:
```
~/.npm-global/lib/node_modules/clawdbot/skills/bird/SKILL.md
```

Check status:
```bash
clawdbot skills info bird
```

---

## Configuration

### Credentials

Bird needs two cookies from X.com:
- `auth_token` — authentication token
- `ct0` — CSRF token

**How to get them:**
1. Log into x.com in browser
2. DevTools (F12) → Application → Cookies → x.com
3. Copy `auth_token` and `ct0` values

### Where Credentials Live

Systemd drop-in (recommended for VPS):
```
~/.config/systemd/user/clawdbot-gateway.service.d/bird.conf
```

Contents:
```ini
[Service]
Environment="AUTH_TOKEN=your_auth_token_here"
Environment="CT0=your_ct0_here"
```

### Updating Credentials

```bash
# Edit the file
nano ~/.config/systemd/user/clawdbot-gateway.service.d/bird.conf

# Reload and restart
systemctl --user daemon-reload
systemctl --user restart clawdbot-gateway
```

---

## Common Commands

```bash
# Check auth
bird whoami

# Read a tweet
bird read https://x.com/user/status/1234567890

# Full thread
bird thread https://x.com/user/status/1234567890

# Your bookmarks
bird bookmarks -n 20

# Search
bird search "from:steipete AI" -n 10

# User's tweets
bird user-tweets @steipete -n 20

# List timeline
bird list-timeline <list-id> -n 20
```

---

## Safety

| Action | Risk |
|--------|------|
| Reading | Low — same as browsing |
| Posting | High — triggers blocks |

**Recommendation:** Use bird for reading only. For posting, use browser automation or pay for X API.

---

## Troubleshooting

**"Missing credentials" error:**
```bash
# Check if env vars are loaded
systemctl --user cat clawdbot-gateway.service | grep AUTH_TOKEN
```

**"Query IDs stale" (404 errors):**
```bash
bird query-ids --fresh
```

**Cookies expired:**
Re-extract from browser and update the drop-in file.

---

## References

- Repo: https://github.com/steipete/bird
- Author: @steipete (Peter Steinberger)
- Skill: `clawdbot skills info bird`
