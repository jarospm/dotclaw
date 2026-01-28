# Server Setup Guide

Generic setup steps for a new Ubuntu VPS (Hetzner, DigitalOcean, etc.)

**Requirements:** 4GB+ RAM recommended for Claude Code

**Recommended specs:** Hetzner CX22/CAX21 or larger, Ubuntu 22.04 LTS

---

## 1. Initial Access

SSH in as root with your key:

```bash
ssh root@<SERVER_IP> -i ~/.ssh/<YOUR_KEY>
```

## 2. Update System

```bash
apt update && apt upgrade -y
```

## 3. Create Deploy User

```bash
# Create user
adduser deploy

# Give sudo powers
usermod -aG sudo deploy

# Copy SSH key from root
rsync --archive --chown=deploy:deploy ~/.ssh /home/deploy/

# Fix permissions
chmod 700 /home/deploy/.ssh
chmod 600 /home/deploy/.ssh/authorized_keys
chown -R deploy:deploy /home/deploy/.ssh

# Lock password (SSH key only)
passwd -l deploy

# Allow passwordless sudo
echo "deploy ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/deploy
chmod 440 /etc/sudoers.d/deploy
```

## 4. Harden SSH

```bash
# Disable password authentication
echo "PasswordAuthentication no" > /etc/ssh/sshd_config.d/99-hardening.conf
systemctl restart ssh
```

## 5. Configure Firewall

```bash
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp
ufw enable
```

---

## 6. Install Stack (as deploy user)

From here, SSH as `deploy` instead of root.

### Node.js (LTS)

**Option A: NodeSource (system-wide, simpler)**

```bash
# Install build deps
sudo apt install -y curl ca-certificates git build-essential

# Add NodeSource repo
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -

# Install
sudo apt install -y nodejs

# Verify
node -v
npm -v

# Enable corepack (for pnpm/yarn)
sudo corepack enable
```

**Option B: nvm (per-user, flexible)**

```bash
sudo apt install -y curl ca-certificates git build-essential
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc
nvm install --lts
node -v && npm -v
```

### Fix npm Global Prefix

Avoids needing sudo for global installs:

```bash
npm config set prefix ~/.npm-global

# Add to ~/.bashrc:
export PATH=~/.npm-global/bin:$PATH

# Reload
source ~/.bashrc
```

### GitHub SSH Key

```bash
# Generate key (empty passphrase)
ssh-keygen -t ed25519 -C "<PURPOSE>-github" -f ~/.ssh/id_ed25519_github -N ""

# Show public key — add to GitHub
cat ~/.ssh/id_ed25519_github.pub

# Configure SSH to use this key
cat >> ~/.ssh/config << 'EOF'
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_github
  IdentitiesOnly yes
EOF
```

---

## 7. Local SSH Config

On your local machine, add to `~/.ssh/config`:

```
Host <ALIAS>-root
  HostName <SERVER_IP>
  User root
  IdentityFile ~/.ssh/<YOUR_KEY>

Host <ALIAS>-deploy
  HostName <SERVER_IP>
  User deploy
  IdentityFile ~/.ssh/<YOUR_KEY>
```

Then connect with just `ssh <ALIAS>-deploy`.

---

## 8. Install Claude Code (optional)

```bash
npm install -g @anthropic-ai/claude-code
claude --version

# Set API key
echo 'export ANTHROPIC_API_KEY=YOUR_KEY_HERE' >> ~/.profile
source ~/.profile
```

## 9. Persistent Sessions with tmux

Keep Claude Code running after you disconnect:

```bash
# Start a new session
tmux new -s claude

# Run claude, then detach with Ctrl+B, D

# Later, reattach
tmux attach -t claude
```

---

## Optional: Tailscale VPN

Add Tailscale if you want to:
- Access the server from a private Tailscale IP (no public SSH exposure)
- Connect multiple devices on a private mesh network

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --ssh

# Lock SSH to VPN only (disables public SSH):
sudo ufw deny 22/tcp
sudo ufw allow in on tailscale0 to any port 22 proto tcp
```

Then connect via Tailscale IP: `ssh deploy@100.x.x.x`

---

## Concepts

### Public IP vs Access

Hetzner gives you a **public IP** — the server is directly on the internet, not behind NAT like your home router.

But **"visible" ≠ "accessible"**:
- **Public IP** = the building address
- **Ports** = doors
- **Services** = what's behind the doors

Unless you run a service and open a port, there's nothing to connect to. With UFW denying incoming by default, your server is a black hole except for the ports you explicitly allow.

### Phone Access (Termius)

You can SSH from your phone using Termius:
- **Host:** Server IP or Tailscale IP
- **Auth:** Import your SSH private key
- Connect → `cd ~/project` → `claude`
