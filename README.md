# Omarchy Desktop on Hetzner Cloud

**Automated deployment of Arch Linux + Hyprland VNC desktop on Hetzner Cloud VPS**

Single-command deployment in ~10 minutes. Zero interaction required.

## Prerequisites

### 1. Install Required Tools

```bash
# macOS
brew install hcloud
brew install hudochenkov/sshpass/sshpass

# Linux
# hcloud
wget https://github.com/hetznercloud/cli/releases/download/v1.42.0/hcloud-linux-amd64.tar.gz
tar -xzf hcloud-linux-amd64.tar.gz
sudo mv hcloud /usr/local/bin/

# sshpass (for Arch ISO key injection)
sudo apt-get install sshpass  # Debian/Ubuntu
sudo yum install sshpass       # RHEL/CentOS
```

**Why sshpass?** The Arch Linux ISO allows password authentication. The script uses `sshpass` to automatically inject your SSH key into the ISO environment.

### 2. Create SSH Key Pair

```bash
# Generate a new SSH key (or use existing)
ssh-keygen -t ed25519 -f ~/.ssh/omarchy_ed25519 -C "omarchy@hetzner"

# This creates:
# ~/.ssh/omarchy_ed25519      (private key - keep secure!)
# ~/.ssh/omarchy_ed25519.pub  (public key - upload to Hetzner)
```

### 3. Add SSH Key to Hetzner

**Via CLI:**
```bash
hcloud ssh-key create --name omarchy-ssh --public-key-from-file ~/.ssh/omarchy_ed25519.pub
```

**Via Web Console:**
1. Go to https://console.hetzner.cloud/
2. Select your project
3. Navigate to Security → SSH Keys
4. Click "Add SSH Key"
5. Paste contents of `~/.ssh/omarchy_ed25519.pub`
6. Name it **exactly** `omarchy-ssh` (script expects this name)

### 4. Get API Token

1. Go to https://console.hetzner.cloud/
2. Select your project
3. Navigate to Security → API Tokens
4. Generate new token with Read & Write permissions
5. Copy the token (you'll only see it once!)

## Quick Start

```bash
# 1. Set your Hetzner API token
export HCLOUD_TOKEN="your-token-here"

# 2. Deploy (uses the 'omarchy-ssh' key you created above)
./deploy.sh
```

**How SSH Authentication Works:**

The script uses your SSH key in three phases:

1. **Server Creation**: Hetzner injects your public key into the temporary Ubuntu image
2. **Arch ISO Phase**: The Arch ISO automatically imports keys from mounted disks
3. **Installed System**: Your key is copied during installation for permanent access

No passwords needed at any stage!

That's it. In ~10 minutes you'll have:
- Arch Linux with LUKS encryption + btrfs
- Hyprland desktop with VNC access
- Auto-boot (no password prompts)

## What You Get

**Minimal Installation** (default):
- Base Arch Linux (196 packages)
- Hyprland compositor
- VNC server (wayvnc)
- Basic waybar status bar
- ~2GB disk usage

**Full Omarchy** (optional):
```bash
INSTALL_OMARCHY=true ./deploy.sh
```
- Everything above PLUS:
- 958 curated packages
- Catppuccin theme
- Custom fonts & icons
- Omarchy menu & scripts
- Full desktop apps (browsers, dev tools, media)

## Configuration

Environment variables (all optional):

```bash
# Hetzner settings
HCLOUD_TOKEN=""              # Required: Your API token
SERVER_NAME="omarchy-123"    # Default: omarchy-<timestamp>
SERVER_TYPE="cpx31"          # Default: 4 vCPU, 8GB RAM, 160GB
SERVER_LOCATION="ash"        # Default: Ashburn, VA

# Credentials
USERNAME="omarchy"           # Default: omarchy
USER_PASSWORD="omarchy123"   # Default: omarchy123
LUKS_PASSWORD="omarchy123"   # Default: omarchy123

# Features
INSTALL_OMARCHY="true"       # Default: false (minimal install)
```

## Access

After deployment completes:

```bash
# VNC
vnc://YOUR_SERVER_IP:5900

# SSH
ssh omarchy@YOUR_SERVER_IP
```

No VNC password by default. Set one if needed:
```bash
ssh omarchy@YOUR_SERVER_IP
wayvncctl set-password  # Set VNC password
```

## Architecture

**Tech Stack:**
- Arch Linux (rolling release)
- LUKS2 encryption with keyfile (auto-unlock)
- btrfs with subvolumes (@, @home, @var, @tmp, @snapshots)
- GRUB bootloader (BIOS mode with GPT)
- Hyprland (Wayland compositor)
- VKMS (virtual GPU for headless rendering)
- wayvnc (VNC server for Wayland)

**Network:**
- Port 22: SSH
- Port 5900: VNC

**Security:**
- LUKS encryption (auto-unlock via keyfile in initramfs)
- SSH key authentication
- Passwordless sudo (development server)

## Costs

Hetzner CPX31:
- €11.90/month (~$13 USD)
- 4 vCPU, 8GB RAM, 160GB NVMe
- 20TB traffic

## Troubleshooting

**VNC shows grey screen:**
```bash
ssh omarchy@YOUR_IP
~/start-desktop.sh  # Restart Hyprland
```

**Check VNC is running:**
```bash
ss -tulnp | grep 5900
```

**View logs:**
```bash
tail -f ~/hyprland.log
tail -f ~/wayvnc.log
```

**Gotchas (all handled by deploy.sh):**
1. **CRITICAL:** `hcloud server reboot` boots from disk, not ISO - must use `shutdown` + `poweron`
2. Verify boot environment to confirm ISO boot (check for `airootfs` or `overlay` filesystem)
3. SSH key injection needed for Arch ISO (requires `sshpass` or manual setup)
4. BIOS boot partition required for GPT + GRUB
5. LUKS keyfile needed for auto-boot
6. VKMS module for headless GPU
7. Monitor name is "Virtual-1" not "HEADLESS-1"
8. wayvnc needs correct output name
9. Environment vars required for Hyprland apps
10. rust/rustup conflict when installing Omarchy

## Manual Steps (if needed)

**Destroy server:**
```bash
hcloud server delete omarchy-123
```

**List servers:**
```bash
hcloud server list
```

**Reboot:**
```bash
hcloud server reboot omarchy-123
```

## Development

Repository structure:
```
omarchy-hetzner-deploy/
├── deploy.sh           # Main deployment script
├── README.md          # This file
└── docs/              # Additional gotchas/notes
```

All deployment logic is in `deploy.sh` for portability.

## Requirements

- `hcloud` CLI: https://github.com/hetznercloud/cli
- SSH key added to Hetzner
- Hetzner API token

## License

MIT

## Credits

- Omarchy: https://github.com/basecamp/omarchy
- Hyprland: https://hyprland.org
- Inspired by: Omakub (https://omakub.org)

---

**Total deployment time:** ~10 minutes
**Lines of code:** ~250
**Human interaction required:** 0
