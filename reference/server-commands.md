# Server Reference

General commands for managing a VPS. See `server-connection.local.md` for actual credentials.

---

## SSH Access

```bash
# As root (initial setup only)
ssh <alias>-root

# As deploy (daily use)
ssh <alias>-deploy
```

Define aliases in `~/.ssh/config`:

```
Host <alias>-root
  HostName <SERVER_IP>
  User root
  IdentityFile ~/.ssh/<YOUR_KEY>

Host <alias>-deploy
  HostName <SERVER_IP>
  User deploy
  IdentityFile ~/.ssh/<YOUR_KEY>
```

---

## Maintenance

Occasionally update packages and reboot:

```bash
# Check for updates
sudo apt update

# Install updates
sudo apt upgrade -y

# Reboot if "System restart required" appears
sudo reboot
```

After reboot, wait ~30-60 seconds then reconnect.

---

## Firewall (UFW)

```bash
# Check status
sudo ufw status

# Allow a port
sudo ufw allow 22/tcp

# Deny a port
sudo ufw deny 22/tcp
```

---

## Service Management (systemd)

Basic commands:

```bash
# Check status
systemctl --user status <service-name>

# View logs (follow)
journalctl --user -u <service-name> -f

# Restart
systemctl --user restart <service-name>

# Stop
systemctl --user stop <service-name>

# Enable lingering (keep services after logout)
loginctl enable-linger $USER
```

**For Clawdbot-specific systemd usage** (drop-ins, adding credentials for skills, etc.), see `systemd.md`.

---

## tmux (Persistent Sessions)

```bash
# Start new session
tmux new -s <name>

# Detach: Ctrl+B, then D

# List sessions
tmux ls

# Reattach
tmux attach -t <name>
```

---

## SSH Tunnels

Forward a remote port to local:

```bash
ssh -N -L <local-port>:127.0.0.1:<remote-port> <alias>-deploy
```

Then access at `http://127.0.0.1:<local-port>/`
