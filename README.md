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

### VNC Grey Screen

**Problem:** VNC client connects but shows only a grey/blank screen

**Root Cause:** Hyprland requires specific environment variables for headless (VNC) operation:
- `WLR_RENDERER_ALLOW_SOFTWARE=1` - Enable software rendering (no GPU)
- `WLR_NO_HARDWARE_CURSORS=1` - Disable hardware cursor
- `WLR_BACKENDS=headless` - Use headless backend for VNC

**Why this happens:** If Omarchy is installed (Phase 4), it uses a modular config system with separate files:
- `~/.config/hypr/envs.conf` - Environment variables
- `~/.config/hypr/monitors.conf` - Monitor configuration
- `~/.config/hypr/autostart.conf` - Startup applications

If these files are empty or missing VNC-specific settings, Hyprland won't render to the virtual display.

**Quick Fix:**
```bash
ssh omarchy@YOUR_IP

# 1. Configure environment for software rendering
cat > ~/.config/hypr/envs.conf <<'EOF'
env = WLR_RENDERER_ALLOW_SOFTWARE,1
env = WLR_NO_HARDWARE_CURSORS,1
env = WLR_BACKENDS,headless
EOF

# 2. Configure virtual monitor
cat > ~/.config/hypr/monitors.conf <<'EOF'
monitor=Virtual-1,1920x1080@60,0x0,1
EOF

# 3. Configure WayVNC autostart
cat > ~/.config/hypr/autostart.conf <<'EOF'
exec-once = sleep 3 && wayvnc --output=Virtual-1 0.0.0.0 5900
EOF

# 4. Restart Hyprland
killall Hyprland wayvnc
export WLR_RENDERER_ALLOW_SOFTWARE=1 WLR_NO_HARDWARE_CURSORS=1 WLR_BACKENDS=headless
sg seat -c "Hyprland > ~/hyprland.log 2>&1 &"
```

**Note:** The deploy script now automatically sets these in Phase 3 and Phase 4 (after Omarchy install), and verifies them in Phase 5.

**Check VNC is running:**
```bash
ss -tulnp | grep 5900
ps aux | grep wayvnc
```

**View logs:**
```bash
tail -f ~/hyprland.log
tail -f ~/wayvnc.log
```

### Known Gotchas (All Handled by deploy.sh)

1. **CRITICAL:** `hcloud server reboot` boots from disk, not ISO - must use `poweroff` + `poweron`
2. **CRITICAL:** `shutdown` is unreliable for ISO boot, use `poweroff` with 15s wait
3. **CRITICAL:** Omarchy config overwrites VNC settings - must recreate after install (Phase 4)
4. Verify boot environment to confirm ISO boot (check for `airootfs` or `overlay` filesystem)
5. SSH key injection can hang when key already works (Hetzner pre-injects via `--ssh-key`)
6. BIOS boot partition required for GPT + GRUB
7. LUKS keyfile needed for auto-boot
8. VKMS module for headless GPU
9. Monitor name is "Virtual-1" not "HEADLESS-1"
10. wayvnc needs correct output name and compositor ready (3s sleep)
11. Environment vars required for Hyprland apps in headless mode
12. rust/rustup conflict when installing Omarchy packages

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
