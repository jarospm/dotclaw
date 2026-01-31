# dotclaw

Personal documentation for running [OpenClaw](https://github.com/openclaw/openclaw) on a VPS.

## Structure

```
core/            # OpenClaw fundamentals
  sessions.md         # Session types, context, compaction
  memory.md           # Memory system and persistence
  auth.md             # Authentication and provider setup
  channels.md         # Telegram and multi-channel setup
  gateway.md          # Gateway service and Control UI
  skills.md           # Skills system and configuration
  structure.md        # File locations, config overview

scheduling/      # Time-based features
  heartbeat.md        # Periodic wake-ups, HEARTBEAT.md
  cron.md             # Scheduled jobs, push notifications

server/          # Infrastructure
  commands.md         # tmux, ufw, SSH tunnels
  systemd.md          # How systemd works with Clawdbot

skills/          # CLI tools
  bird.md             # X/Twitter (bundled skill + CLI)
  firecrawl.md        # Web research (custom skill + CLI)

integrations/    # External services
  moltbook.md         # Agent social network

guides/          # First-time setup walkthroughs
  server-setup.md     # VPS setup (Ubuntu, Node, SSH, UFW)
  openclaw-setup.md   # OpenClaw installation & onboarding
```

## Quick Links

| Want to... | Go to |
|------------|-------|
| Understand sessions | `core/sessions.md` |
| Set up reminders | `scheduling/cron.md` |
| Configure heartbeat | `scheduling/heartbeat.md` |
| Add a skill | `core/skills.md` |
| Use bird/X | `skills/bird.md` |
| Use Moltbook | `integrations/moltbook.md` |
| Debug systemd | `server/systemd.md` |
