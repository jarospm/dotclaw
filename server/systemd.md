# Systemd for Clawdbot

Systemd is Linux's service manager. It handles starting, stopping, and managing long-running processes.

---

## Why Clawdbot Uses Systemd

Without systemd:
- You'd manually start the gateway every reboot
- You'd need a terminal open running the process
- If it crashes, it stays dead

With systemd:
- Gateway starts automatically on boot
- Restarts automatically on crash
- Runs in background (no terminal needed)
- Logs are captured centrally

---

## User vs System Services

Clawdbot runs as a **user service** (not system-wide):

```bash
# User service (what Clawdbot uses)
systemctl --user status clawdbot-gateway

# System service (NOT this)
sudo systemctl status some-service
```

User services:
- Run as your user (deploy), not root
- Config lives in `~/.config/systemd/user/`
- Need `loginctl enable-linger` to run without active login

---

## Service Files

Main service file (managed by Clawdbot — don't edit):
```
~/.config/systemd/user/clawdbot-gateway.service
```

Contains:
- `ExecStart` — command to run
- `Restart` — restart policy (always, on-failure)
- `Environment` — environment variables

View it:
```bash
cat ~/.config/systemd/user/clawdbot-gateway.service
```

---

## Drop-in Files (Extending Services)

To add configuration **without editing the main file**, use drop-ins:

```
~/.config/systemd/user/clawdbot-gateway.service.d/
└── bird.conf           # Your additions
└── another-skill.conf  # More additions
```

Systemd merges these with the main service file.

**Why drop-ins?**
- Clawdbot updates won't overwrite your customizations
- Clean separation of concerns
- Easy to add/remove skill configurations

---

## Adding Environment Variables for Skills

Many skills need API keys or credentials. The gateway process is the parent of all agent exec calls, so env vars set on the gateway are inherited by skills.

**Example: Adding bird (X/Twitter) credentials**

```bash
# Create drop-in directory
mkdir -p ~/.config/systemd/user/clawdbot-gateway.service.d/

# Create drop-in file
cat > ~/.config/systemd/user/clawdbot-gateway.service.d/bird.conf << 'EOF'
[Service]
Environment="AUTH_TOKEN=your_auth_token_here"
Environment="CT0=your_ct0_here"
EOF

# Reload systemd and restart gateway
systemctl --user daemon-reload
systemctl --user restart clawdbot-gateway
```

**Example: Adding another skill's API key**

```bash
cat > ~/.config/systemd/user/clawdbot-gateway.service.d/some-skill.conf << 'EOF'
[Service]
Environment="SOME_API_KEY=xxx"
Environment="ANOTHER_SECRET=yyy"
EOF

systemctl --user daemon-reload
systemctl --user restart clawdbot-gateway
```

---

## How It All Connects

```
systemd
  └── clawdbot-gateway.service (+ drop-ins merged)
        │
        │  Environment: AUTH_TOKEN, CT0, SOME_API_KEY, ...
        │
        └── Gateway process (Node.js)
              │
              └── Agent exec calls
                    │
                    └── bird, other CLIs (inherit env vars)
```

When the agent runs `bird read <url>`:
1. Gateway spawns a shell process
2. Shell inherits gateway's environment
3. `bird` reads `AUTH_TOKEN` and `CT0` from environment
4. Authentication works

---

## Common Commands

```bash
# Check status
systemctl --user status clawdbot-gateway

# View logs (live)
journalctl --user -u clawdbot-gateway -f

# View recent logs
journalctl --user -u clawdbot-gateway -n 50

# Restart (after config changes)
systemctl --user daemon-reload
systemctl --user restart clawdbot-gateway

# Stop
systemctl --user stop clawdbot-gateway

# Start
systemctl --user start clawdbot-gateway

# Check what drop-ins are loaded
systemctl --user cat clawdbot-gateway.service
```

---

## Troubleshooting

**Service won't start after reboot?**
```bash
loginctl enable-linger $USER
```

**Env vars not working?**
```bash
# Check if drop-in is loaded
systemctl --user cat clawdbot-gateway.service

# Look for your Environment lines
```

**Gateway keeps crashing?**
```bash
# Check logs for errors
journalctl --user -u clawdbot-gateway -n 100
```

---

## File Locations Summary

```
~/.config/systemd/user/
├── clawdbot-gateway.service                    # Main (managed by Clawdbot)
└── clawdbot-gateway.service.d/                 # Your drop-ins
    ├── bird.conf                               # X/Twitter credentials
    └── other-skill.conf                        # Other skill credentials
```
