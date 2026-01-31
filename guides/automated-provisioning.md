# Automated Server Provisioning

Create new clawdbot servers from your Mac using hcloud CLI + cloud-init.

**What this automates:** Everything in `server-setup.md` steps 1-6 — system updates, deploy user, SSH hardening, firewall, Node.js installation.

**What remains manual:** GitHub SSH key (per-instance), clawdbot onboarding (interactive).

---

## Prerequisites (Mac)

### 1. Install hcloud CLI

```bash
brew install hcloud
```

### 2. Create API Token

1. Go to [Hetzner Cloud Console](https://console.hetzner.cloud/)
2. Select your project (or create one)
3. Security → API Tokens → Generate API Token
4. Name it (e.g., "provisioning") and give Read & Write access
5. Copy the token — you won't see it again

### 3. Configure hcloud

```bash
hcloud context create clawdbot
# Paste your API token when prompted
```

Verify:
```bash
hcloud server list
```

### 4. Register Your SSH Key

Your Mac's SSH key must be registered with Hetzner:

```bash
# Upload your key (use your actual key path)
hcloud ssh-key create --name macbook --public-key-from-file ~/.ssh/id_ed25519.pub

# Verify
hcloud ssh-key list
```

---

## Current Server Specs (Reference)

This doc is based on the existing clawdbot server:

- **Server type:** CAX11 (ARM, 2 vCPU, 4GB RAM, ~€4/month)
- **Image:** Ubuntu 24.04 LTS
- **Location:** fsn1 (Falkenstein) — or nbg1, hel1
- **Node.js:** v24 LTS via NodeSource
- **Architecture:** aarch64 (ARM)

---

## Cloud-Init Template

Save this as `cloud-init.yaml` on your Mac.

It automates:
- System update
- Deploy user creation with sudo
- SSH hardening (password auth disabled)
- UFW firewall (SSH only)
- Node.js 24 LTS installation
- npm global prefix setup
- corepack enabled (for pnpm/yarn)

```yaml
#cloud-config

# Create deploy user with sudo access
users:
  - name: deploy
    groups: sudo
    shell: /bin/bash
    sudo: ALL=(ALL) NOPASSWD:ALL
    lock_passwd: true
    ssh_authorized_keys:
      - <YOUR_SSH_PUBLIC_KEY>

# System packages
package_update: true
package_upgrade: true
packages:
  - curl
  - ca-certificates
  - git
  - build-essential
  - ufw
  - tmux

# Write config files
write_files:
  # SSH hardening
  - path: /etc/ssh/sshd_config.d/99-hardening.conf
    content: |
      PasswordAuthentication no
    permissions: "0644"

  # npm global prefix for deploy user
  - path: /home/deploy/.npmrc
    content: |
      prefix=/home/deploy/.npm-global
    owner: deploy:deploy
    permissions: "0644"

  # PATH additions for deploy user
  - path: /home/deploy/.bashrc.d/path.sh
    content: |
      export PATH="$HOME/.npm-global/bin:$PATH"
      export PATH="$HOME/.local/bin:$PATH"
    owner: deploy:deploy
    permissions: "0644"

  # Source .bashrc.d scripts
  - path: /home/deploy/.bashrc.d/.loader
    content: |
      # This file is sourced by .bashrc additions below
    owner: deploy:deploy
    permissions: "0644"

# Run commands after boot
runcmd:
  # Firewall setup
  - ufw default deny incoming
  - ufw default allow outgoing
  - ufw allow 22/tcp
  - ufw --force enable

  # Restart SSH to apply hardening
  - systemctl restart ssh

  # Install Node.js 24 LTS
  - curl -fsSL https://deb.nodesource.com/setup_24.x | bash -
  - apt-get install -y nodejs

  # Enable corepack
  - corepack enable

  # Create npm-global directory
  - mkdir -p /home/deploy/.npm-global
  - chown deploy:deploy /home/deploy/.npm-global

  # Create .bashrc.d directory
  - mkdir -p /home/deploy/.bashrc.d
  - chown deploy:deploy /home/deploy/.bashrc.d

  # Add .bashrc.d sourcing to .bashrc
  - |
    echo '' >> /home/deploy/.bashrc
    echo '# Source custom scripts' >> /home/deploy/.bashrc
    echo 'for f in ~/.bashrc.d/*.sh; do [ -r "$f" ] && . "$f"; done' >> /home/deploy/.bashrc

  # Fix ownership of deploy home
  - chown -R deploy:deploy /home/deploy
```

**Important:** Replace `<YOUR_SSH_PUBLIC_KEY>` with your actual public key.

Get your public key:
```bash
cat ~/.ssh/id_ed25519.pub
```

---

## Create a Server

### One-liner

```bash
hcloud server create \
  --name clawd-02 \
  --type cax11 \
  --image ubuntu-24.04 \
  --location fsn1 \
  --ssh-key macbook \
  --user-data-from-file cloud-init.yaml
```

### Parameters

- **--name** — unique server name (clawd-01, clawd-02, etc.)
- **--type** — server size:
  - `cax11` — ARM, 2 vCPU, 4GB RAM, ~€4/mo (recommended)
  - `cax21` — ARM, 4 vCPU, 8GB RAM, ~€7/mo (for browser automation)
  - `cx22` — x86, 2 vCPU, 4GB RAM, ~€6/mo (if ARM causes issues)
- **--image** — `ubuntu-24.04` (LTS)
- **--location** — datacenter:
  - `fsn1` — Falkenstein, Germany
  - `nbg1` — Nuremberg, Germany
  - `hel1` — Helsinki, Finland
- **--ssh-key** — name of your registered SSH key
- **--user-data-from-file** — path to cloud-init.yaml

### Output

```
Server 12345678 created
IPv4: 123.45.67.89
```

---

## Post-Provision Steps

Cloud-init takes ~2-3 minutes to complete. Then SSH in as deploy:

```bash
ssh deploy@<SERVER_IP>
```

### 1. Verify Setup

```bash
node -v          # Should show v24.x
npm -v           # Should show 11.x
sudo ufw status  # Should show SSH only
```

### 2. Generate GitHub SSH Key

```bash
ssh-keygen -t ed25519 -C "clawd-02-github" -f ~/.ssh/id_ed25519_github -N ""
cat ~/.ssh/id_ed25519_github.pub
# Add this to GitHub → Settings → SSH Keys

# Configure SSH
cat >> ~/.ssh/config << 'EOF'
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_github
  IdentitiesOnly yes
EOF

# Test
ssh -T git@github.com
```

### 3. Install Clawdbot

```bash
npm install -g clawdbot@latest
clawdbot --version
```

### 4. Run Onboarding

```bash
clawdbot onboard --install-daemon
```

Follow the interactive prompts for:
- AI provider (setup-token recommended)
- Gateway bind/auth
- Channel tokens (Telegram, Discord, etc.)

---

## Local SSH Config

Add to `~/.ssh/config` on your Mac for easy access:

```
Host clawd-02
  HostName <SERVER_IP>
  User deploy
  IdentityFile ~/.ssh/id_ed25519
```

Then connect with just: `ssh clawd-02`

---

## Automation Script (Optional)

For spinning up multiple servers, create `provision.sh`:

```bash
#!/bin/bash
set -e

NAME="$1"
if [ -z "$NAME" ]; then
  echo "Usage: ./provision.sh <server-name>"
  exit 1
fi

echo "Creating server: $NAME"

hcloud server create \
  --name "$NAME" \
  --type cax11 \
  --image ubuntu-24.04 \
  --location fsn1 \
  --ssh-key macbook \
  --user-data-from-file cloud-init.yaml

echo ""
echo "Server created. Wait ~2-3 min for cloud-init, then:"
echo "  ssh deploy@\$(hcloud server ip $NAME)"
```

Usage:
```bash
chmod +x provision.sh
./provision.sh clawd-03
```

---

## Teardown

Delete a server:
```bash
hcloud server delete clawd-02
```

List all servers:
```bash
hcloud server list
```

---

## Troubleshooting

**Cloud-init didn't complete:**
```bash
# SSH as root first
ssh root@<SERVER_IP>

# Check cloud-init status
cloud-init status
cat /var/log/cloud-init-output.log
```

**Node not found after SSH:**
Cloud-init may still be running. Wait 2-3 minutes, or check:
```bash
tail -f /var/log/cloud-init-output.log
```

**Wrong architecture:**
ARM (cax*) is cheaper but some npm packages may not have ARM builds.
Switch to x86 (cx22) if you hit native module issues.

---

## What's NOT Automated

These require per-instance manual setup:

- **GitHub SSH key** — each server needs its own key added to GitHub
- **Clawdbot onboarding** — interactive wizard for tokens and personality
- **Claude setup-token** — requires browser auth
- **Tailscale** — optional, run `tailscale up --ssh` if needed

---

## Cost Reference

| Type  | vCPU | RAM  | Monthly |
|-------|------|------|---------|
| cax11 | 2    | 4GB  | ~€4     |
| cax21 | 4    | 8GB  | ~€7     |
| cax31 | 8    | 16GB | ~€14    |
| cx22  | 2    | 4GB  | ~€6     |

ARM (cax*) is ~30% cheaper than x86 (cx*) for same specs.
