# Zero Trust Home Security Lab

## Overview
A personal cybersecurity lab built on a Raspberry Pi 5 to develop hands-on experience with Zero Trust network architecture, secure remote access, DNS filtering, and access logging. Every configuration decision is intentional and documented with the security reasoning behind it.

**Hardware:** Raspberry Pi 5  
**OS:** Raspberry Pi OS Lite (64-bit) — Debian GNU/Linux 13 (Trixie)  
**Access Method:** SSH over ethernet (headless — no monitor, no keyboard)  
**Status:** Phase 1 complete — base OS configured and hardened

---

## Phase 1: OS Installation & Base Configuration

### Objectives
- Deploy a hardened, headless Linux server
- Enable SSH access with non-default credentials
- Disable unused network interfaces to reduce attack surface
- Establish a clean, documented baseline for future IAM and network security tooling

---

### Step 1 — Flashing the OS

**Tool used:** Raspberry Pi Imager  
**OS selected:** Raspberry Pi OS Lite (64-bit)

Lite was chosen deliberately over the full desktop version for two reasons. First, real-world IAM infrastructure runs on headless servers without a GUI — practicing in this environment mirrors production conditions. Second, every piece of software running on a system is a potential vulnerability. A desktop environment, browser, and file manager are all unnecessary components that expand the attack surface without adding value to the lab.

**Pre-configured in Imager before flashing:**
- Custom hostname: `securitypilab`
- Custom username (non-default — the old default `pi` username is publicly known and a common attack target)
- Strong password
- WiFi credentials (for initial connectivity)
- SSH enabled with password authentication
- Raspberry Pi Connect disabled (see rationale below)
- Locale set to `America/Vancouver`

**Why pre-configure at flash time:** Baking credentials and SSH into the image before first boot reduces the window of exposure. The Pi is never online with default credentials.

---

### Step 2 — First Boot & SSH Connection

The Pi was booted headless — no monitor or keyboard attached. Access was established entirely through SSH from a Windows laptop.

**What SSH is:** Secure Shell is a protocol that allows encrypted remote access and control of another computer over a network using the command line. It is the standard method for accessing servers in production environments.

**Why SSH over Raspberry Pi Connect:** Raspberry Pi Connect routes traffic through Raspberry Pi's cloud relay servers. In a Zero Trust model, the goal is to minimize external dependencies and keep access control in-house. SSH over a local network — and later over a Tailscale mesh VPN — keeps the access path fully controlled without relying on a third-party intermediary.

**Connection method:**
```bash
ssh username@<internal-ip>
```

**Finding the Pi's IP address:** The router admin page was unreachable on this network, so the Pi was located using the Windows command line. `ipconfig` identified the local network range, `arp -a` listed recently seen devices, and `ping` confirmed which were live Linux systems (TTL=64 is the Linux default). SSH was then attempted against each candidate IP to identify the Pi by which one accepted the connection on port 22.

**Known_hosts conflict:** A fingerprint mismatch warning appeared on first connection because the same IP address had been used by a previous Pi setup. SSH stores a cryptographic fingerprint of every device it connects to in a `known_hosts` file. When the OS was reflashed, the Pi generated a new fingerprint — which didn't match the saved one. This is SSH's protection against man-in-the-middle attacks, where a bad actor positions themselves between a client and server, intercepting traffic while impersonating both sides. The stale entry was cleared with `ssh-keygen -R` and the new legitimate fingerprint was accepted.

```bash
ssh-keygen -R <internal-ip>
```

---

### Step 3 — System Updates

First action after connecting — update all packages.

```bash
sudo apt update && sudo apt upgrade -y
```

**Command breakdown:**
- `sudo` — "Superuser do." Executes the command with administrator privileges. This reflects the principle of least privilege: the system doesn't grant admin rights by default, they must be explicitly invoked when needed.
- `apt` — Debian's package manager. The equivalent of an app store for the command line.
- `update` — Refreshes the list of available packages without installing anything.
- `&&` — Logical gate. The upgrade only runs if the update succeeded.
- `upgrade -y` — Installs newer versions of all packages. The `-y` flag auto-confirms all prompts.

---

### Step 4 — System Verification

**OS version:**
```bash
cat /etc/os-release
```
Output confirmed: `Debian GNU/Linux 13 (Trixie)` — a widely used enterprise Linux base.

`/etc` is the standard directory for system-wide configuration files on any Linux system. Checking `/etc/os-release` before doing anything else on an unfamiliar server is standard practice — never assume what you're working on.

**Network interfaces:**
```bash
ip addr show
```

Three interfaces were present:
- `lo` — Loopback. A virtual interface every Linux system has. The address `127.0.0.1` always means "this machine talking to itself." Used for internal processes, never a real network connection.
- `eth0` — Ethernet, assigned `192.168.1.x`. Active.
- `wlan0` — WiFi, assigned `192.168.1.x`. Active — but unnecessary.

**Disk usage:**
```bash
df -h
```

| Partition | Size | Used | Available | Use% |
|-----------|------|------|-----------|------|
| /dev/mmcblk0p2 (main) | 117G | 4.1G | 109G | 4% |
| /dev/mmcblk0p1 (boot) | 505M | 65M | 440M | 13% |

Baseline state: fresh install, 4% used, 109GB available. `mmcblk` is Linux's designation for SD card storage (MultiMediaCard).

---

### Step 5 — Disabling WiFi

With ethernet active and confirmed working, the WiFi interface serves no purpose. Every active network interface is a potential attack surface. Leaving WiFi running creates an unnecessary second entry point into the system with no operational benefit.

**Attack surface reduction** is a core Zero Trust principle: disable everything that isn't needed. The Pi should have one controlled, auditable path in and out — the ethernet cable.

```bash
sudo nmcli radio wifi off
sudo systemctl disable wpa_supplicant
```

**What these commands do:**
- `nmcli radio wifi off` — Disables the WiFi interface via NetworkManager (the network management service on modern Debian systems).
- `systemctl disable wpa_supplicant` — Prevents the WiFi authentication service from starting on boot. Without this step, `wpa_supplicant` would attempt to re-enable WiFi on every reboot.

**Verification:**
```bash
nmcli radio
```

| WIFI-HW | WIFI | WWAN-HW | WWAN |
|---------|------|---------|------|
| enabled | disabled | missing | enabled |

WiFi hardware is present but disabled in software. No IP address assigned to `wlan0`. Confirmed persistent across reboot.

---

### Step 6 — Reboot & Verify

```bash
sudo reboot
```

**Important:** Always use a clean shutdown command — never unplug a running Linux system. Cutting power while the filesystem is mounted can corrupt the SD card.

After reboot, SSH reconnected successfully over ethernet at `192.168.1.x`. WiFi remained disabled. Single controlled access path confirmed.

---

## Phase 1 Exit State

| Component | Status |
|-----------|--------|
| OS | Raspberry Pi OS Lite 64-bit (Debian 13 Trixie) |
| Hostname | securitypilab |
| Access | SSH over ethernet only |
| WiFi | Disabled (software + boot service) |
| Packages | Fully updated |
| Attack surface | Minimized — single network interface active |

---

## Key Concepts Demonstrated

**Principle of Least Privilege** — Admin rights (`sudo`) invoked only when required, not running as root by default.

**Attack Surface Reduction** — Headless OS (no GUI), WiFi disabled, Raspberry Pi Connect not enabled. Every unnecessary component removed.

**Zero Trust Access Model** — Single, controlled, auditable network path. No third-party relay services. SSH authentication required for all access.

**Man-in-the-Middle Protection** — SSH `known_hosts` fingerprint verification. Demonstrated in practice when a fingerprint conflict was encountered and resolved with `ssh-keygen -R`.

---

## Next: Phase 2 — Tailscale Mesh VPN
*Installing Tailscale to establish secure remote access from anywhere without exposing the Pi to the public internet.*
