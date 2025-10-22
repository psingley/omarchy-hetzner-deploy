# Next Session: Complete Omarchy Deployment

## Context
You're continuing work on automated Omarchy deployment to Hetzner Cloud. Previous session created a **validated base deployment** (Arch + Hyprland + VNC) in a single automated script.

**What's Done:**
- ✅ Single automated deployment script (`deploy.sh`)
- ✅ Base Arch + LUKS + btrfs + GRUB (auto-boot)
- ✅ Hyprland + VNC working (validated at 178.156.187.8:5900)
- ✅ 18 gotchas solved and automated
- ✅ GitHub repo: https://github.com/psingley/omarchy-hetzner-deploy

**What's NOT Done:**
- ❌ Full Omarchy installation (958 packages) not tested yet
- ❌ Omarchy themes, fonts, icons not validated
- ❌ Blog post / documentation

## Your Task

### 1. Validate the Recipe (20-30 min)

Deploy a **fresh instance** using our automated script to confirm it works:

```bash
# You're already in the repo: /Users/psingley/omarchy-hetzner-deploy

# Set credentials (already should have these)
export HCLOUD_TOKEN="aeRLfddNYRr9I3SfrU9ByOYyC4gWGzxKiU5w2u3SOmFh2Ggz8NRyPh7Ud6OjAMgh"

# Deploy MINIMAL version first (validate base works)
./deploy.sh

# This should:
# - Create new Hetzner server
# - Install Arch + Hyprland + VNC
# - Complete in ~10 minutes
# - Output VNC connection info
```

**Validation Checklist:**
- [ ] Script runs without errors
- [ ] Server created successfully
- [ ] VNC accessible (test the connection!)
- [ ] Hyprland desktop visible with waybar
- [ ] No grey screen issues

**If anything fails:** Document what broke, check against the 18 known gotchas in session context.

### 2. Complete Full Omarchy Installation (30-45 min)

Once minimal deployment is validated, test the **full Omarchy** installation:

```bash
# Deploy with full Omarchy
INSTALL_OMARCHY=true ./deploy.sh

# This will:
# - Do everything from step 1
# - Clone Omarchy repo
# - Install 958 packages (15-20 min)
# - Configure themes, fonts, icons
# - Apply all gotchas #13-17
```

**Validation Checklist:**
- [ ] All 958 packages installed
- [ ] VNC shows Catppuccin theme
- [ ] Icons visible in waybar
- [ ] Omarchy menu works (Super+Space)
- [ ] Custom font installed
- [ ] Wallpaper visible

**Document any new issues** - we solved 18 gotchas, there may be more!

### 3. Refine & Document (15-20 min)

Based on your findings:

**If everything works:**
- Update README with confirmed timings
- Add screenshots (optional)
- Mark as "production ready"

**If you found issues:**
- Fix the `deploy.sh` script
- Document new gotchas (Gotcha #19+)
- Test fix on another fresh instance
- Commit improvements

### 4. Blog Post Prompt (10 min)

Create `BLOG_POST_PROMPT.md` with:
- Deployment story (what we built)
- Technical highlights
- All 18+ gotchas as learning moments
- Performance metrics
- Call to action

## Key Information

**Previous Session Servers:**
- 178.156.187.8 - Fresh validation server (minimal Hyprland + VNC working)
- 178.156.192.214 - Earlier test server (may still be running)

**Known Working:**
- Base Arch installation
- LUKS auto-unlock (keyfile in initramfs)
- Hyprland with VKMS headless rendering
- VNC via wayvnc on port 5900
- Monitor name: "Virtual-1" (not "HEADLESS-1")

**Gotchas to Watch For:**
1. BIOS boot partition required (automated)
2. LUKS keyfile for auto-boot (automated)
3. VKMS module loading (automated)
4. SSH keys in installed system (automated)
5-17. See session history
18. VNC monitor names (Virtual-1 not HEADLESS-1)

**Credentials:**
- LUKS: omarchy123
- User: omarchy / omarchy123
- Root: omarchy123

## Questions You Might Have

**Q: Can I clean up old test servers?**
A: Yes! List with `hcloud server list`, delete with `hcloud server delete <name>`

**Q: Should I modify deploy.sh during testing?**
A: YES! Fix any issues you find, then commit improvements.

**Q: What if the full Omarchy install fails?**
A: Common issues:
- rust/rustup conflict (should be handled by dual `yes` streams)
- Timeouts (may need to increase ssh timeout)
- Missing dependencies (add to deploy.sh)

**Q: How do I test VNC quickly?**
A: `ssh omarchy@<IP> 'ss -tulnp | grep 5900'` then connect with VNC client

**Q: Git account for commits?**
A: Use `git account personal` before committing. Repo is on psingley (not psingley_collette).

## Success Criteria

By end of session:
- ✅ Fresh deployment from `deploy.sh` works perfectly
- ✅ Full Omarchy (958 packages) validated
- ✅ VNC shows beautiful themed desktop
- ✅ Any issues found are fixed and documented
- ✅ Ready to share publicly

## Start Here

1. Read this file completely
2. Export HCLOUD_TOKEN
3. Run `./deploy.sh` (minimal first!)
4. Test VNC connection
5. If successful, try full Omarchy
6. Document findings
7. Ask questions if blocked!

**Time budget:** 60-90 minutes total

---

**Previous session achieved:** Minimal Hyprland VNC in 10 minutes, fully automated
**Your goal:** Validate + complete full Omarchy installation
