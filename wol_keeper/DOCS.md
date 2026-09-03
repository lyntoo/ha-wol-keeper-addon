# WoL Keeper

Keeps Wake-on-LAN (magic packet) enabled on a network interface across reboots and clean shutdowns.

## The problem this solves

Several onboard NIC drivers — most commonly **Realtek `r8169`** — reset Wake-on-LAN to "disabled" on every boot, regardless of what's set in the BIOS/UEFI. The BIOS enables it, the driver disables it again a few seconds later. Every single boot. This add-on runs in the background and re-enables it automatically.

## Before you install

This add-on cannot help if the hardware/BIOS itself blocks Wake-on-LAN:
- Enable **Wake-on-LAN** in the BIOS/UEFI (usually under Power Management)
- Some boards also need "Deep Sleep"/"ErP"/"EuP" disabled, or "Enable UEFI Network Stack" left on

Check with `ethtool <interface>` (e.g. via the Advanced SSH & Web Terminal add-on) — if it shows `Supports Wake-on: g` but `Wake-on: d`, this add-on is for you.

## Configuration

| Option | Description | Default |
|---|---|---|
| `interface` | Network interface name (e.g. `eth0`, `enp2s0`). Leave blank for auto-detection of the first physical wired interface. | `""` (auto) |
| `check_interval` | Seconds between checks. If Wake-on-LAN is found disabled, it's re-enabled immediately. | `300` |

## Checking it's working

Open the add-on's **Log** tab. On startup it logs the interface it's using and whether Wake-on-LAN capability was detected. Every `check_interval` seconds (and immediately on start), it logs whether it had to re-enable the setting.

## Uninstalling

Stop the add-on, then use the **⋮** menu → **Uninstall**. It doesn't modify anything on the host filesystem directly — everything it does is scoped to the network interface's driver-level setting via `ethtool`, which is not persistent storage; nothing to clean up beyond removing the add-on itself.

## Known limitations

- A true full power-cut (not just a clean OS shutdown) still depends on the NIC regaining standby power from the BIOS/PSU before it can respond to a magic packet — this add-on cannot control that.
- If the BIOS/UEFI doesn't support Wake-on-LAN at all, this add-on cannot add that capability — it only keeps an *existing* capability from being disabled by the OS/driver.
