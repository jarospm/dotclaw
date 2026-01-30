# dotclaw

Personal documentation for running [OpenClaw](https://github.com/openclaw/openclaw) on a VPS.

## Structure

```
guides/          # First-time setup walkthroughs
  server-setup.md     # VPS setup (Ubuntu, Node, SSH, UFW)
  openclaw-setup.md   # OpenClaw installation & onboarding

reference/       # Day-to-day operations
  server-commands.md     # tmux, ufw, SSH tunnels
  systemd.md             # How systemd works with Clawdbot (important!)
  openclaw-structure.md  # File locations, config overview
  openclaw-auth.md       # Authentication and provider setup
  openclaw-gateway.md    # Gateway service and Control UI
  openclaw-channels.md   # Telegram and multi-agent setup
  openclaw-memory.md     # Memory system and persistence
  openclaw-skills.md     # Skills system and configuration

tools/           # Individual tool documentation
  bird.md             # X/Twitter CLI (reading tweets, bookmarks)
  firecrawl.md        # Web search, scrape, map (replaces built-in web tools)

local/           # Instance-specific (gitignored)
```

## Usage

- **New server?** Start with `guides/server-setup.md`
- **Installing OpenClaw?** Follow `guides/openclaw-setup.md`
- **Managing existing bot?** See `reference/`
- **Your specific config?** Check `local/`
