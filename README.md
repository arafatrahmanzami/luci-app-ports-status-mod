<div align="center">

# luci-app-ports-status-mod

**Enhanced LuCI port status module for OpenWrt / ImmortalWrt — custom labels, descriptions, drag-and-drop ordering, live status dots, port enable/disable, and full DSA + swconfig + single-port auto-detection.**

[![GitHub release](https://img.shields.io/github/v/release/arafatrahmanzami/luci-app-ports-status-mod?style=flat-square&color=blue)](https://github.com/arafatrahmanzami/luci-app-ports-status-mod/releases)
[![GitHub stars](https://img.shields.io/github/stars/arafatrahmanzami/luci-app-ports-status-mod?style=flat-square&color=yellow)](https://github.com/arafatrahmanzami/luci-app-ports-status-mod/stargazers)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)
[![OpenWrt](https://img.shields.io/badge/OpenWrt-24.10%20%7C%2025.12-blue?style=flat-square&logo=openwrt)](https://openwrt.org)
[![Platform](https://img.shields.io/badge/Platform-aarch64%20%7C%20mips%20%7C%20arm-lightgrey?style=flat-square)]()

</div>

---

## Features

- **Custom labels** — click any port card to rename it
- **Descriptions** — add a note under each port
- **Drag-and-drop** — reorder port cards by holding and dragging
- **Live status dots**:
  - 🟢 Green (blinking) — port is active, traffic flowing
  - 🟠 Orange — idle, link up but no traffic
  - ⚫ Gray — down, no link
  - 🔴 Red — disabled by the user
- **Enable / Disable** LAN ports from the UI
- **Backup / Restore** — export and import config as JSON
- **Persistent config** at `/etc/user_defined_ports.json`
- **Fast cached backend** — 0.01s response (vs 1.18s originally)
- **Auto-detect** — works on DSA, swconfig, and single-port devices
- **Zone color bars** — shows firewall zone per port
- **Speed indicator** — 1 GbE / 100 M / etc.
- **RX/TX counters** — per-port traffic with hover tooltip for detailed stats

## Auto-Detect Architecture

One package works on every OpenWrt device:

| Architecture | Devices | Support |
|---|---|---|
| **DSA** (Distributed Switch Architecture) | MediaTek filogic, Qualcomm, modern targets | ✅ Full |
| **swconfig** | Older ath79, ar71xx, ramips | ✅ Full |
| **Single-port** (no switch chip) | TP-Link TL-WA1201, travel routers | ✅ Full |

## Compatibility

| OpenWrt Version | Package Format | Status |
|---|---|---|
| 24.10.x | `.ipk` (opkg) | ✅ Tested |
| 25.12.x | `.apk` (apk) | ✅ Tested |
| Snapshots | `.apk` | ✅ Likely works |

**Targets tested:** `mediatek/filogic` (aarch64_cortex-a53), `ath79/generic` (mips_24kc).


## Installation

### From GitHub Release (recommended)

OpenWrt 24.10.x / ImmortalWrt (IPK):

    opkg install luci-app-ports-status-mod_*.ipk

OpenWrt 25.12.x (APK):

    apk add --allow-untrusted luci-app-ports-status-mod-*.apk

### From Source

    git clone https://github.com/arafatrahmanzami/luci-app-ports-status-mod.git
    cp -r luci-app-ports-status-mod ~/openwrt-sdk/package/
    cd ~/openwrt-sdk
    make package/luci-app-ports-status-mod/compile V=s

For APK format, use the OpenWrt 25.12 SDK with the same commands.

## After Installation

Clear LuCI cache and restart the web server:

    rm -rf /tmp/luci-*
    /etc/init.d/uhttpd restart

Then hard-refresh the browser (Ctrl+Shift+R or Cmd+Shift+R). Navigate to Status -> Overview. The enhanced Port status section should appear.

## Usage

### Rename a Port

1. Click any port label (for example eth1)
2. Enter a new label (max 9 chars) and optional description (max 50 chars)
3. Click Save

### Reorder Ports

Click and hold a port card for about 300ms (600ms on touch), then drag to the desired position.

### Enable or Disable a Port

1. Click a LAN port label
2. Toggle Enable port or Disable port
3. Click Save

The dot turns red when disabled.

### Backup and Restore

From any port edit modal:

- Create backup .bak - saves config to /etc/user_defined_ports.json.bak
- Save .json file - downloads to your computer
- Upload .json file - restores from a file
- Restore backup .bak - restores the .bak file

## Configuration

Port labels persist in /etc/user_defined_ports.json:

    [
      {
        "device": "eth1",
        "label": "eth1=WAN1",
        "role": "wan",
        "originalLabel": "eth1",
        "description": "wanpppoe/wandhcp"
      },
      {
        "device": "lan1",
        "label": "lan1=WAN2",
        "role": "lan",
        "originalLabel": "lan1",
        "description": "wanbdhcp/wanbpppoe"
      }
    ]

To reset to defaults, delete the file and reload LuCI:

    rm /etc/user_defined_ports.json
    rm -rf /tmp/luci-*
    /etc/init.d/uhttpd restart


## Uninstall

### OpenWrt 24.10.x (IPK)

Remove the package:

    opkg remove luci-app-ports-status-mod

### OpenWrt 25.12.x (APK)

    apk del luci-app-ports-status-mod

### Restore Stock LuCI Port Widget

The package installs its JS over the stock LuCI file and backs up the original as /www/luci-static/resources/view/status/include/29_ports.bak. To restore:

    cp /www/luci-static/resources/view/status/include/29_ports.bak \
       /www/luci-static/resources/view/status/include/29_ports.js
    rm -rf /tmp/luci-*
    /etc/init.d/uhttpd restart

Then hard-refresh the browser.

### Full Cleanup

To remove all traces including custom config:

    opkg remove luci-app-ports-status-mod   # or: apk del luci-app-ports-status-mod
    rm -f /etc/user_defined_ports.json /etc/user_defined_ports.json.bak
    rm -f /etc/ports_status
    rm -rf /tmp/.ports_status_cache /tmp/.ports_prev_counters
    rm -rf /tmp/luci-*
    /etc/init.d/uhttpd restart

## File Layout

After installation on the router:

    /www/luci-static/resources/view/status/include/
        29_ports.js        -> active (our custom file, copied on install)
        29_ports.bak       -> stock file (auto-backed up once)

    /usr/share/luci-app-ports-status-mod/
        29_ports.js        -> source of our custom UI (not web-accessible)

    /usr/libexec/rpcd/
        ports-status-mod   -> RPC backend (DSA/swconfig auto-detect)

    /usr/share/rpcd/acl.d/
        luci-app-ports-status-mod.json  -> ACL permissions

    /usr/bin/
        toggle_ports.sh    -> enable/disable the mod on demand

    /etc/uci-defaults/
        98-install-ports-mod      -> swaps custom JS into place on install
        99-cleanup-ports-symlink  -> safe symlink cleanup

    /etc/
        user_defined_ports.json       -> your labels and descriptions
        user_defined_ports.json.bak   -> automatic backup
        ports_status                  -> state file for disabled ports

## How It Works

The RPC backend auto-detects the switch architecture at runtime:

- DSA: reads /sys/class/net/* and /proc/net/dev, measures traffic delta between calls
- swconfig: parses output of swconfig dev switchN show
- Single-port: reads the one physical interface directly

The backend caches results for 3 seconds to avoid CPU spikes when multiple browser tabs are open. Each cache hit costs under 0.05s versus the original 1.18s.

## Troubleshooting

### Port status section is missing

Hard-refresh the browser (Ctrl+Shift+R). If still missing, clear the LuCI cache:

    rm -rf /tmp/luci-*
    /etc/init.d/uhttpd restart

### Dots show Unknown

Restart rpcd:

    killall rpcd
    /etc/init.d/rpcd start

### Dots do not blink

Blinking only happens when a port has actual traffic flowing. If all ports are idle, no blinking is expected. Generate traffic (a download or ping) and check again.

### Duplicate Port status section

A leftover file from an older version is being loaded. Remove it:

    rm -f /www/luci-static/resources/view/status/include/29_ports_custom.js
    rm -rf /tmp/luci-*
    /etc/init.d/uhttpd restart

### Config is lost after reboot

Check that /etc/user_defined_ports.json is a real file, not a symlink:

    ls -la /etc/user_defined_ports.json

If it shows -> /tmp/... it will be cleared on reboot. Recreate it:

    rm /etc/user_defined_ports.json
    touch /etc/user_defined_ports.json
    chmod 666 /etc/user_defined_ports.json

### Install fails with file conflict

If installing on an older system:

    opkg install luci-app-ports-status-mod_*.ipk --force-overwrite

Or on APK:

    apk add --allow-untrusted luci-app-ports-status-mod-*.apk

## Changelog

See CHANGELOG.md for the full version history.

## Credits

Original concept: 4IceG/luci-app-ports-status-mod
This fork: Arafat Rahman Zami - auto-detection, swconfig support, single-port fallback, cached backend, clean packaging.

## License

MIT (c) 2026 Arafat Rahman Zami

## Port Details Modal

Click the small circular **i** button in the top-right of any port card to open a detailed view:

- Link speed (Mb/s), duplex, operstate, carrier, MTU, MAC
- SFP module diagnostics (temperature, voltage, TX bias, TX/RX power) — shown only on SFP-capable ports
- PoE status detection — shown only on PoE-capable hardware
- ethtool hardware statistics — errors, dropped packets, collisions (requires `ethtool` package)

## Real-Time Speed

Each port card shows live Rx/Tx rate:

    ▲ 1.2 MB/s
    ▼ 47.6 MB/s

Updates every 3 seconds. The value is cached in the browser so it never flashes `--` during LuCI re-renders.

To install the optional `ethtool` package for extended hardware stats:

    # OpenWrt 24.10.x
    opkg install ethtool

    # OpenWrt 25.12.x
    apk add ethtool

