# dotmolt

Personal documentation for running [Moltbot](https://github.com/moltbot/moltbot) on a VPS.

## Structure

```
guides/          # First-time setup walkthroughs
  server-setup.md    # VPS setup (Ubuntu, Node, SSH, UFW)
  moltbot-setup.md   # Moltbot installation & onboarding

reference/       # Day-to-day operations
  server-commands.md    # systemd, tmux, ufw, SSH tunnels
  moltbot-structure.md  # File locations, config overview
  moltbot-auth.md       # Authentication and provider setup
  moltbot-gateway.md    # Gateway service and Control UI
  moltbot-channels.md   # Telegram and multi-agent setup

local/           # Instance-specific (gitignored)
```

## Usage

- **New server?** Start with `guides/server-setup.md`
- **Installing Moltbot?** Follow `guides/moltbot-setup.md`
- **Managing existing bot?** See `reference/`
- **Your specific config?** Check `local/`
