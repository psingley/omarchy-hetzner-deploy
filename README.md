# Omarchy Desktop on Hetzner Cloud

**Automated deployment of Arch Linux + Hyprland VNC desktop on Hetzner Cloud VPS**

Single-command deployment in ~10 minutes. Zero interaction required.

## Quick Start

```bash
# 1. Get Hetzner API token from https://console.hetzner.cloud/
export HCLOUD_TOKEN="your-token-here"

# 2. Create SSH key in Hetzner (name it 'omarchy-ssh')

# 3. Deploy
./deploy.sh
```

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
1. BIOS boot partition required for GPT + GRUB
2. LUKS keyfile needed for auto-boot
3. VKMS module for headless GPU
4. Monitor name is "Virtual-1" not "HEADLESS-1"
5. wayvnc needs correct output name
6. Environment vars required for Hyprland apps
7. rust/rustup conflict when installing Omarchy

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
