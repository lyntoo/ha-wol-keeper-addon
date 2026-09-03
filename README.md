# Home Assistant WoL Keeper Add-on

[![Open your Home Assistant instance and show the add add-on repository dialog with a specific repository URL pre-filled.](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Flyntoo%2Fha-wol-keeper-addon)

Keeps **Wake-on-LAN (magic packet)** enabled on a network interface, even after a clean shutdown or a normal reboot — for **Home Assistant OS (HAOS)** running directly on physical hardware (bare metal), not inside a VM.

---

## Who this is for

You installed HAOS on a physical PC (Dell, HP, generic mini-PC, NUC, etc.), enabled **Wake-on-LAN in the BIOS/UEFI**, confirmed it's the correct setting — and it still doesn't work: a magic packet sent after a shutdown never wakes the machine back up.

This is a known issue with several onboard NIC drivers (most commonly **Realtek `r8169`** chipsets, very common on desktop motherboards) that **silently reset Wake-on-LAN to "disabled" at every boot**, ignoring the BIOS setting entirely. The BIOS enables it; the Linux driver disables it again a few seconds later, on every single boot. No amount of BIOS tweaking fixes this — it has to be re-applied in the OS after every boot.

**This add-on is for you if:**
- You run **HAOS bare-metal** (not a VM — inside a VM, the hypervisor's virtual NIC usually doesn't have this problem)
- You've confirmed Wake-on-LAN is enabled in the BIOS/UEFI (`Power Management` → `Wake on LAN`, or similar)
- `ethtool <interface>` (via the Advanced SSH & Web Terminal add-on, or similar) shows `Supports Wake-on: g` but `Wake-on: d` — i.e., the card supports magic packet but it's currently disabled
- A magic packet sent right after boot works — but stops working after a reboot or clean shutdown

**Not needed if:**
- You run HA in a VM (Proxmox, ESXi, etc.) — fix this at the hypervisor/virtual NIC level instead
- Your NIC already keeps Wake-on-LAN enabled across reboots on its own (some drivers behave correctly)

## Supported systems

- **Home Assistant OS**, bare metal, `amd64` / `aarch64` / `armv7`
- Any physical Ethernet NIC supported by `ethtool` — confirmed working on **Realtek r8169** (the most common affected chipset); should work on any NIC that reports `Supports Wake-on: g` under `ethtool`
- **Not applicable** to Wi-Fi-only setups (Wake-on-LAN requires a wired Ethernet connection) or to Supervised/Container installations sharing a host that already manages this itself

---

## Installation

1. Click the badge above, or manually go to **Settings → Add-ons → Add-on Store → ⋮ (top right) → Repositories**, and add:
   `https://github.com/lyntoo/ha-wol-keeper-addon`
2. Refresh the Add-on Store page — **"WoL Keeper"** will appear under the new repository.
3. Click it, then **Install**.
4. Go to the **Configuration** tab and set your options (see below) — leave `interface` blank for auto-detection.
5. Start the add-on, and enable **"Start on boot"**.

## Configuration

| Option | Description | Default |
|---|---|---|
| `interface` | Network interface name (e.g. `eth0`, `enp2s0`). Leave blank to auto-detect the first physical wired interface. | `""` (auto) |
| `check_interval` | How often (in seconds) to re-check and re-enable Wake-on-LAN if it's found disabled. | `300` |

Check the add-on **Log** tab after starting — it reports the detected/configured interface and confirms each time it re-enables Wake-on-LAN.

## Uninstalling

**Settings → Add-ons → WoL Keeper → Stop**, then the **⋮** menu → **Uninstall**. No files are left behind outside the add-on's own data — nothing is written to the host filesystem directly, and no other add-on or HA configuration is touched.

## Prerequisites (must be done first, outside this add-on)

This add-on **cannot** enable Wake-on-LAN if the hardware itself doesn't support it or the BIOS blocks it:
- Enable **Wake-on-LAN** in the BIOS/UEFI (often under Power Management, sometimes per-NIC as "LAN" vs "WLAN")
- Some systems also require disabling a "Deep Sleep" / "ErP" / "EuP" power-saving mode, or need "Enable UEFI Network Stack" left on — check your specific motherboard/BIOS documentation if magic packets still don't wake the machine after installing this add-on.

## How it works

Runs continuously in the background: every `check_interval` seconds, it checks the interface's Wake-on-LAN status via `ethtool`, and if it finds it disabled (`d` instead of `g`), it re-enables magic-packet mode (`ethtool -s <interface> wol g`) — including immediately on the add-on's own startup, right after every boot.

---

Not affiliated with any NIC manufacturer. Provided as-is — test on your own hardware before relying on it.
