🌐 English

# luci-app-ports-status-mod

A lightweight LuCI plugin that upgrades the boring **Port status** widget in OpenWrt / ImmortalWrt into a fully customizable live dashboard — custom labels, drag-and-drop ordering, real-time speed, per-port details, and auto-detection of DSA / swconfig / single-port switches.

[![GitHub release](https://img.shields.io/github/v/release/arafatrahmanzami/luci-app-ports-status-mod?style=flat-square&color=blue)](https://github.com/arafatrahmanzami/luci-app-ports-status-mod/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)
[![OpenWrt](https://img.shields.io/badge/OpenWrt-24.10%20%7C%2025.12-blue?style=flat-square&logo=openwrt)](https://openwrt.org)

Release: 1.0.14 — 2026-10-05 — by @arafatrahmanzami

Built on top of the original concept by 4IceG, extended with auto-detection for every switch architecture, a fast cached backend, real-time speed, and a full port details modal. Tested on OpenWrt and ImmortalWrt 24.10 and 25.12. It is a pure frontend + small shell backend — no kernel changes, no risk to the boot.

## What does this actually do?

Imagine the Port status section on your router's Status → Overview page. By default it shows a fixed row of cards labeled eth0, lan1, lan2, lan3 — with no way to rename them, no speed info, and no details.

This plugin replaces that widget with one that:

- Lets you **rename** every port ("Office PC", "NAS", "AP-cable", "WAN-fiber")
- Lets you **reorder** cards by dragging them
- Shows **live Rx/Tx speed** under each port
- Shows a **status dot** (green = active, orange = idle, gray = down, red = disabled)
- Provides a **details modal** with speed, duplex, MAC, MTU, and (when available) SFP and PoE info
- Persists all your customizations across reboots
- Works on **every OpenWrt device** — DSA switches, older swconfig, or even single-Ethernet-port travel routers

## You need this if:

✅ You have a router with multiple Ethernet ports and want to identify them at a glance
✅ You want to see live upload/download speed per port
✅ You want a clean, labeled view of your WAN / LAN / trunk ports
✅ You want to enable or disable individual LAN ports from the UI
✅ You want to see hardware details (link speed, duplex, MAC) of each port

## You don't need this if:

❌ You have only one Ethernet port and don't care about labels
❌ You never look at the Status → Overview page
❌ You're happy with the stock OpenWrt port widget

## Table of Contents

- [What does this actually do?](#what-does-this-actually-do)
- [Why Port Status Mod?](#why-port-status-mod)
- [Key Features](#key-features)
- [Prerequisites](#prerequisites)
- [Before You Start](#before-you-start)
- [Installation](#installation)
- [After Installation](#after-installation)
- [Quick Start](#quick-start)
- [Detailed Usage](#detailed-usage)
- [Port Details Modal](#port-details-modal)
- [Real-Time Speed](#real-time-speed)
- [Configuration](#configuration)
- [Uninstall](#uninstall)
- [File Layout](#file-layout)
- [How It Works](#how-it-works)
- [Troubleshooting](#troubleshooting)
- [Changelog](#changelog)
- [Compile from Source](#compile-from-source)
- [Credits](#credits)
- [Glossary](#glossary)
- [License](#license)

## Why Port Status Mod?

The default OpenWrt Port status widget works fine for a stock router — but as soon as you have a router with 4+ Ethernet ports, a WAN trunk, a wired mesh backbone, or a NAS plugged into lan3, the anonymous "lan1 / lan2 / lan3 / lan4" labels become useless.

This plugin solves that. It was written to make home and small-office port management actually readable — you look at the Status page, and instantly know which physical port is doing what.

Built-in advantages:

- ✅ Works on **all switch architectures** — DSA, swconfig, single-port
- ✅ Zero kernel changes — pure shell backend + LuCI JavaScript
- ✅ Zero boot risk — if the plugin fails, LuCI simply falls back to the stock widget
- ✅ Fully reversible — stock file is backed up as 29_ports.bak on install
- ✅ Persists across reboots — labels live in /etc/user_defined_ports.json
- ✅ Fast — cached backend response in 0.01s (was 1.18s in the original)
- ✅ Auto-detects switch type — no config needed

## Key Features

### Custom Labels and Descriptions

Click any port card to open the edit modal. Change the label (max 9 chars) and add an optional description (max 50 chars). Changes save immediately and persist across reboots.

### Drag-and-Drop Ordering

Click and hold any port card, drag it to a new position, release. Order is saved automatically. Works from anywhere on the card — the icon, the label, or the description area.

### Live Status Dots

Each port shows a colored dot to indicate its state:

- 🟢 Green (blinking) — Active: traffic flowing
- 🟠 Orange — Idle: link up, but no traffic
- ⚫ Gray — Down: no link
- 🔴 Red — Disabled: user disabled it from the UI

### Real-Time Rx/Tx Speed

Below each port card you see the live throughput:

    ▲ 1.2 MB/s
    ▼ 47.6 MB/s

Updates every 3 seconds. Smoothed so it doesn't flicker. Survives LuCI re-renders.

### Port Details Modal

Click the small "i" circle in the top-right of any port card to see:

- Link speed (Mb/s)
- Duplex (full / half)
- Operstate (up / down / unknown)
- Carrier (present / absent)
- MTU
- MAC address
- SFP module diagnostics — temperature, voltage, TX bias, TX/RX power
- PoE status — where supported
- ethtool hardware stats — errors, dropped packets, collisions (requires ethtool package)

### Enable / Disable Ports

For LAN ports (not WAN), you can enable or disable the port directly from the UI. Useful when you want to isolate a device or save power.

### Backup / Restore

From any port's edit modal:

- **Create backup .bak** — saves config to /etc/user_defined_ports.json.bak
- **Save .json file** — downloads the config to your computer
- **Upload .json file** — restores from a file
- **Restore backup .bak** — restores the .bak file

### Auto-Detect Architecture

The backend automatically detects the switch type at runtime:

| Architecture | How it's detected | Devices |
|---|---|---|
| DSA | Reads /sys/class/net + /proc/net/dev | MediaTek filogic, Qualcomm, modern targets |
| swconfig | Parses swconfig dev switchN show | Older ath79, ar71xx, ramips |
| Single-port | Reads the one physical interface | Travel routers, TL-WA1201, etc. |

No configuration is needed — the plugin figures it out.

## Prerequisites

To function correctly, the app requires:

- OpenWrt / ImmortalWrt 24.10 or 25.12
- LuCI web interface (installed by default)
- luci-mod-status (installed by default)

Optional:

- ethtool package — for extended hardware stats in the details modal
- SFP-capable hardware — for SFP module temperature and power readings
- PoE-capable hardware (realtek-poe) — for PoE status

## Before You Start

Answer these five questions before installing:

1. Do I have SSH or LuCI access to my router? — You'll need one of them.
2. Do I know my router's IP address? — Usually 192.168.1.1 or 192.168.10.1.
3. Do I have a working internet connection on the router? — To download the package.
4. Have I backed up my current config? — In LuCI: System → Backup / Flash Firmware → Generate archive.
5. Do I know my architecture? — Not needed — the package is architecture-independent (PKGARCH=all).

If all five are ✅, proceed. If any is ❌, fix that first.

## Installation

This plugin is `PKG_ARCH=all` — no architecture detection needed. Works on every router CPU (x86, MIPS, ARM, AArch64, RISC-V).

### Option A — Install from GitHub Release (recommended)

**OpenWrt 24.10.x / ImmortalWrt (opkg, `.ipk`):**

    cd /tmp && \
    wget https://github.com/arafatrahmanzami/luci-app-ports-status-mod/releases/download/v1.0.14/luci-app-ports-status-mod_1.0.14-r1_all.ipk && \
    opkg install luci-app-ports-status-mod_1.0.14-r1_all.ipk

**OpenWrt 25.12.x (apk, `.apk`):**

    cd /tmp && \
    wget https://github.com/arafatrahmanzami/luci-app-ports-status-mod/releases/download/v1.0.14/luci-app-ports-status-mod-1.0.14-r1.apk && \
    apk add --allow-untrusted luci-app-ports-status-mod-1.0.14-r1.apk

After install, restart rpcd and uhttpd:

    rm -f /tmp/luci-indexcache /tmp/luci-modulecache/*
    /etc/init.d/rpcd restart
    /etc/init.d/uhttpd restart

### Option B — Single auto-detect command (opkg vs apk)

Run this one-liner. It picks `apk` or `opkg` automatically based on which is present.

    cd /tmp && \
    if command -v apk >/dev/null 2>&1; then \
      echo "Detected apk — OpenWrt 25.12+" && \
      wget -O pkg.apk https://github.com/arafatrahmanzami/luci-app-ports-status-mod/releases/download/v1.0.14/luci-app-ports-status-mod-1.0.14-r1.apk && \
      apk add --allow-untrusted pkg.apk; \
    else \
      echo "Detected opkg — OpenWrt 24.10 or older" && \
      opkg update && \
      wget -O pkg.ipk https://github.com/arafatrahmanzami/luci-app-ports-status-mod/releases/download/v1.0.14/luci-app-ports-status-mod_1.0.14-r1_all.ipk && \
      opkg install pkg.ipk; \
    fi && \
    rm -f /tmp/luci-indexcache /tmp/luci-modulecache/* && \
    /etc/init.d/rpcd restart && \
    /etc/init.d/uhttpd restart

### Option C — Offline install (no internet on the router)

Copy the `.ipk` / `.apk` file to the router's `/tmp` folder using WinSCP, FileZilla, or any SFTP manager.

**2a — OpenWrt ≤ 24.10 (opkg):**

    cd /tmp
    opkg install luci-app-ports-status-mod_1.0.14-r1_all.ipk
    /etc/init.d/uhttpd restart

**2b — OpenWrt ≥ 25.12 (apk):**

    cd /tmp
    apk --allow-untrusted add luci-app-ports-status-mod-1.0.14-r1.apk
    /etc/init.d/uhttpd restart

### Option D — Build from source

See the [Compile from Source](#compile-from-source) section below.

## After Installation

Open LuCI in your browser:

    http://<router-ip>/cgi-bin/luci/

Navigate to **Status → Overview**. The Port status section should now show your customized view.

If the menu doesn't show or you see the old stock widget:

1. Hard-refresh your browser: **Ctrl + Shift + R** (or **Cmd + Shift + R** on Mac)
2. If still stale, clear the LuCI cache:
   
       rm -f /tmp/luci-indexcache /tmp/luci-modulecache/*
       /etc/init.d/uhttpd restart

3. Reload in a new Incognito window to rule out browser cache

### What gets installed

| Path | Purpose |
|---|---|
| /www/luci-static/resources/view/status/include/29_ports.js | Active custom JS (installed over stock) |
| /www/luci-static/resources/view/status/include/29_ports.bak | Backup of stock LuCI file |
| /usr/share/luci-app-ports-status-mod/29_ports.js | Source of the custom UI (not web-accessible) |
| /usr/libexec/rpcd/ports-status-mod | Main RPC backend (state / port status) |
| /usr/libexec/rpcd/ports-status-mod-ext | Extension RPC backend (speed, details, SFP, PoE) |
| /usr/share/rpcd/acl.d/luci-app-ports-status-mod.json | ACL permissions for the RPC methods |
| /usr/bin/toggle_ports.sh | Manual enable / disable script |
| /etc/uci-defaults/98-install-ports-mod | Installs custom JS over stock on first install |
| /etc/uci-defaults/99-cleanup-ports-symlink | Safe cleanup of a symlink if present |
| /etc/user_defined_ports.json | Your labels and descriptions (created on first use) |
| /etc/ports_status | State file tracking user-disabled ports |

## Quick Start

After install, go to **Status → Overview** and look at the Port status section.

**Rename a port (30 seconds):**

1. Click any port label — for example `eth1`
2. The Edit Port Label modal opens
3. Enter a new label (max 9 chars) and optional description (max 50 chars)
4. Click **Save**

The change appears instantly on the card and is written to `/etc/user_defined_ports.json`.

**Reorder ports (10 seconds):**

1. Click and hold any port card — anywhere on the card
2. Move the mouse sideways by about 10 pixels
3. The card lifts and follows your cursor
4. Drop it where you want it
5. Release — the new order saves automatically

**Enable or disable a LAN port:**

1. Click a LAN port label (not WAN)
2. In the modal, toggle **Enable port** or **Disable port**
3. Click **Save**

The dot turns **red** when disabled and the port is added to `/etc/ports_status`.

## Detailed Usage

### Editing labels and descriptions

Every port card supports two editable fields:

- **Label** — displayed in bold at the top of the card. Max 9 characters to prevent overflow. Defaults to the port name (e.g. `eth1`).
- **Description** — a smaller line below the label. Max 50 characters. Optional but useful for noting what's plugged in (e.g. `wanpppoe/wandhcp`, `br-lan+lan_fallback`).

### Working with the status dot

| Dot color | Meaning | What to do |
|---|---|---|
| 🟢 Green, blinking | Active — traffic flowing | Nothing — normal state |
| 🟠 Orange | Idle — link up, no traffic | Plug a device in or generate traffic |
| ⚫ Gray | Down — no link | Check the cable |
| 🔴 Red | Disabled — user turned it off | Click the label, re-enable |

### Enabling / disabling ports

Only LAN ports can be disabled — this is intentional (disabling the WAN port from LuCI would disconnect you). The plugin tracks disabled ports in `/etc/ports_status`. To re-enable, click the label and toggle **Enable port**.

### Backup and restore

From any port edit modal you have four options:

| Button | What it does |
|---|---|
| **Create backup .bak** | Copies current config to `/etc/user_defined_ports.json.bak` |
| **Save .json file** | Downloads current config as a `.json` file |
| **Upload .json file** | Restores from a `.json` file on your PC |
| **Restore backup .bak** | Restores the `.bak` file created earlier |

Backup files are read-only (chmod 444) to prevent accidental corruption.

## Port Details Modal

Click the small circular **i** button in the top-right corner of any port card to open the details modal.

**Basic info:**

| Field | Example | Source |
|---|---|---|
| Interface | `eth1` | port name |
| Link speed | `1000 Mb/s` | /sys/class/net/eth1/speed |
| Duplex | `full` | /sys/class/net/eth1/duplex |
| Operstate | `up` | /sys/class/net/eth1/operstate |
| Carrier | `present` | /sys/class/net/eth1/carrier |
| MTU | `1492` | /sys/class/net/eth1/mtu |
| MAC | `90:6a:94:12:a6:28` | /sys/class/net/eth1/address |

**SFP Module** — shown only if the port has an SFP transceiver with DDM support:

- Temperature (°C)
- Voltage (V)
- TX bias current (mA)
- TX power (dBm)
- RX power (dBm)

**PoE Status** — shown only if the hardware exposes PoE info:

- Detection / class / power reading
- Realtek-poe aware — if `realtek-poe` ubus service is present, uses that
- Otherwise falls back to `ethtool --show-poe`

**ethtool Hardware Stats** — requires the `ethtool` package:

    # OpenWrt 24.10.x
    opkg install ethtool

    # OpenWrt 25.12.x
    apk add ethtool

Once installed, the modal shows errors, dropped packets, collisions, and more.

## Real-Time Speed

Each port card shows live Rx/Tx throughput:

    ▲ 1.2 MB/s
    ▼ 47.6 MB/s

- Updates every **3 seconds**
- Cached in the browser — **never flashes back to `--`** during LuCI re-renders
- Idle ports show `0 B/s` (not blank)
- Non-physical interfaces (WLAN, bridges, VPN) are excluded from the display

The rate is calculated by the backend comparing `/proc/net/dev` counters over a short interval and dividing by the elapsed time — no CPU-intensive polling.

## Configuration

### Config file

All user customizations live in `/etc/user_defined_ports.json`:

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

### Editing the config manually

If you prefer to edit the JSON directly:

    nano /etc/user_defined_ports.json
    chmod 444 /etc/user_defined_ports.json   # keep read-only
    rm -rf /tmp/luci-*
    /etc/init.d/uhttpd restart

### Resetting to defaults

To wipe all custom labels and start fresh:

    rm -f /etc/user_defined_ports.json /etc/user_defined_ports.json.bak
    rm -f /etc/ports_status
    rm -rf /tmp/luci-*
    /etc/init.d/uhttpd restart

Next time you open Status → Overview, the plugin regenerates defaults from `board.json`.

### Disabling the plugin temporarily

The `toggle_ports.sh` script swaps between the custom UI and the stock UI:

    /usr/bin/toggle_ports.sh    # run once to disable, run again to enable

This is useful for debugging. It preserves your labels in `/etc/user_defined_ports.json`.

## Uninstall

### OpenWrt 24.10.x (IPK)

    opkg remove luci-app-ports-status-mod

### OpenWrt 25.12.x (APK)

    apk del luci-app-ports-status-mod

### Restore stock LuCI port widget

The package installs its JS over the stock LuCI file and backs up the original as `/www/luci-static/resources/view/status/include/29_ports.bak`. To restore the original:

    cp /www/luci-static/resources/view/status/include/29_ports.bak \
       /www/luci-static/resources/view/status/include/29_ports.js
    rm -rf /tmp/luci-*
    /etc/init.d/uhttpd restart

Then hard-refresh the browser.

### Full cleanup

To remove every trace, including custom config:

    opkg remove luci-app-ports-status-mod   # or: apk del luci-app-ports-status-mod
    rm -f /etc/user_defined_ports.json /etc/user_defined_ports.json.bak
    rm -f /etc/ports_status
    rm -rf /tmp/.ports_status_cache /tmp/.ports_prev_counters /tmp/.ps_speed_*
    rm -rf /tmp/luci-*
    /etc/init.d/uhttpd restart

## File Layout

After installation:

    /www/luci-static/resources/view/status/include/
        29_ports.js         <- active (our custom file, installed over stock)
        29_ports.bak        <- stock LuCI file (auto-backed up once)

    /usr/share/luci-app-ports-status-mod/
        29_ports.js         <- source of the custom UI (not web-accessible)

    /usr/libexec/rpcd/
        ports-status-mod    <- main RPC backend (state / port status)
        ports-status-mod-ext <- extension RPC backend (speed, details, SFP, PoE)

    /usr/share/rpcd/acl.d/
        luci-app-ports-status-mod.json  <- ACL permissions

    /usr/bin/
        toggle_ports.sh     <- enable / disable the custom UI on demand

    /etc/uci-defaults/
        98-install-ports-mod      <- installs custom JS over stock on install
        99-cleanup-ports-symlink  <- safe symlink cleanup

    /etc/
        user_defined_ports.json       <- your labels and descriptions
        user_defined_ports.json.bak   <- automatic backup
        ports_status                  <- state file for disabled ports

    /tmp/  (ephemeral, wiped on reboot)
        .ports_status_cache     <- 3-second cache for main backend
        .ports_prev_counters    <- previous snapshot for active / idle detection
        .ps_speed_ts            <- timestamp for live speed calculation
        .ps_speed_prev          <- previous counter snapshot for live speed

## How It Works

### Architecture detection

At runtime, the extension backend detects the switch architecture:

| Detected as | How | Result |
|---|---|---|
| DSA | swconfig not present, /sys/class/net has real ports | Reads /proc/net/dev + /sys/class/net per port |
| swconfig | `swconfig list` returns a device | Parses `swconfig dev switchN show` output |
| Single-port | No switch, only one physical interface | Reads the single interface directly |

No user configuration is required.

### Backend caching

The main `getPortsStatus` method is cached for **3 seconds**. This means:

- The first call after a page load takes ~0.01s
- Repeated calls within 3s return the cached result instantly
- Multiple browser tabs do not multiply backend calls

This is what fixed the original CPU spike issue.

### Live speed calculation

Rather than polling every second, the backend compares two counter snapshots:

1. Reads current byte counters from `/proc/net/dev`
2. Compares to the previous snapshot (stored in `/tmp/.ps_speed_prev`)
3. Divides the delta by the elapsed time (from `/tmp/.ps_speed_ts`)
4. Returns bytes/second

This runs in <10ms and uses no background daemon.

### Frontend polling

The JS uses a **single global interval** (`setInterval` running every 3 seconds) to fetch speed updates. Results are cached in module-scope variables so that when LuCI re-renders the Port status section, the new cards are populated instantly from the cache — no `--` flash.

## Troubleshooting

### Port status section is missing

Hard-refresh the browser: **Ctrl + Shift + R**. If still missing, clear LuCI cache:

    rm -f /tmp/luci-indexcache /tmp/luci-modulecache/* /tmp/luci-*cache*
    /etc/init.d/uhttpd restart

### Dots show "Unknown"

The RPC backend isn't responding. Restart rpcd:

    killall rpcd
    sleep 2
    /etc/init.d/rpcd start

Then hard-refresh the browser.

### Dots do not blink

Blinking only happens when a port has **actual traffic flowing**. If all ports are idle, no blinking is expected. Generate traffic (a download or a ping) and check again.

### Duplicate Port status section

A leftover file from an older version is being loaded. Remove it:

    rm -f /www/luci-static/resources/view/status/include/29_ports_custom.js
    rm -rf /tmp/luci-*
    /etc/init.d/uhttpd restart

### Config is lost after reboot

Check that `/etc/user_defined_ports.json` is a real file, not a symlink:

    ls -la /etc/user_defined_ports.json

If the output shows `-> /tmp/...` then the file will be cleared on reboot. Recreate it:

    rm /etc/user_defined_ports.json
    touch /etc/user_defined_ports.json
    chmod 666 /etc/user_defined_ports.json

### Install fails with file conflict

If installing on an older system:

    opkg install luci-app-ports-status-mod_*.ipk --force-overwrite

Or on APK:

    apk add --allow-untrusted luci-app-ports-status-mod-*.apk

### Speed line shows `--` or doesn't appear

The extension backend may not be running. Verify it's registered:

    ubus -v list ports-status-mod-ext

If nothing appears, reinstall the package. If the object is registered but speed doesn't show, run:

    ubus call ports-status-mod-ext getPortSpeed

You should see JSON with per-port rates.

### Drag-and-drop doesn't work

Make sure you're dragging the **card itself** (not the small "i" info button). Hold the mouse down and move sideways by at least 6-10 pixels — this triggers the drag handler. If nothing happens:

- Clear browser cache (Ctrl + Shift + R)
- Check that the JS file is the latest: `wc -c /www/luci-static/resources/view/status/include/29_ports.js` should return **51588** or higher

### Editing labels doesn't work

Click the label **without moving the mouse**. If you move more than 6 pixels, the card will start dragging instead. This is by design.

### CPU spikes after installing

The original 4IceG version caused CPU spikes because each call spawned ~25 shell processes. This fork caches results for 3 seconds and uses a single `/proc/net/dev` read per call. If you still see spikes:

    top -b -d 2 -n 15 | grep -E "PID|rpcd|ports-status"

If `ports-status-mod` processes appear in the output, wait 30 seconds and re-check. The cache should kick in.

### Uninstall leaves leftover files

Run the "Full cleanup" section above. This removes everything including the state file.

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for the full version history.

Quick version summary:

- **1.0.14** — Real-time speed, Port Details modal, SFP + PoE detection, better drag
- **1.0.13** — Clean install on both formats, fixed duplicate Port status section
- **1.0.12** — Fixed file conflict with luci-mod-status
- **1.0.11** — swconfig + single-port fallback
- **1.0.10** — Auto-detect DSA vs swconfig
- **1.0.9** — Fixed RPC backend list case
- **1.0.8** — Initial fork from 4IceG

## Compile from Source

Uses the standard OpenWrt SDK. No custom toolchain needed.

### Step 1 — Download the SDK

Pick the SDK that matches your target and OpenWrt version.

**For OpenWrt 24.10.x (IPK):**

    wget https://downloads.openwrt.org/releases/24.10.6/targets/mediatek/filogic/openwrt-sdk-24.10.6-mediatek-filogic_gcc-13.3.0_musl.Linux-x86_64.tar.zst
    tar --use-compress-program=unzstd -xf openwrt-sdk-24.10.6-mediatek-filogic_gcc-13.3.0_musl.Linux-x86_64.tar.zst
    mv openwrt-sdk-24.10.6-mediatek-filogic_* openwrt-sdk

**For OpenWrt 25.12.x (APK):**

    wget https://downloads.openwrt.org/releases/25.12.5/targets/ath79/generic/openwrt-sdk-25.12.5-ath79-generic_gcc-14.3.0_musl.Linux-x86_64.tar.zst
    tar --use-compress-program=unzstd -xf openwrt-sdk-25.12.5-ath79-generic_gcc-14.3.0_musl.Linux-x86_64.tar.zst
    mv openwrt-sdk-25.12.5-ath79-generic_* openwrt-sdk-ath79

### Step 2 — Add the package source

    cd openwrt-sdk
    mkdir -p package/luci-app-ports-status-mod
    cp -a /path/to/luci-app-ports-status-mod/. package/luci-app-ports-status-mod/

Or clone directly:

    cd package
    git clone https://github.com/arafatrahmanzami/luci-app-ports-status-mod.git

### Step 3 — Install feeds

    cd openwrt-sdk
    ./scripts/feeds update -a
    ./scripts/feeds install -a

### Step 4 — Build

    make defconfig
    echo "CONFIG_PACKAGE_luci-app-ports-status-mod=m" >> .config
    make defconfig
    make package/luci-app-ports-status-mod/compile V=s

Output for IPK:

    bin/packages/aarch64_cortex-a53/base/luci-app-ports-status-mod_*_all.ipk

Output for APK:

    bin/packages/mips_24kc/base/luci-app-ports-status-mod-*.apk

## Credits

- Original concept: **4IceG** — https://github.com/4IceG/luci-app-ports-status-mod
- Current fork and maintainer: **Arafat Rahman Zami** — [@arafatrahmanzami](https://github.com/arafatrahmanzami)

Extended with:

- Auto-detection of DSA / swconfig / single-port switches
- Fast cached backend (0.01s response, down from 1.18s)
- Real-time Rx/Tx speed display
- Port Details modal with SFP + PoE support
- Rewritten drag-and-drop using pointer events
- Clean install on both IPK and APK formats
- GitHub Actions CI building both formats on every tag

## Glossary

Terms used in this README that may be unfamiliar, especially if you're new to OpenWrt or networking.

### Network fundamentals

| Term | Full form | Meaning |
|---|---|---|
| LAN | Local Area Network | The network inside your home or office |
| WAN | Wide Area Network | The wider network — usually the internet |
| IP | Internet Protocol | Each device's unique address (e.g. 192.168.1.1) |
| MAC | Media Access Control | Each network card's unique hardware ID (e.g. 90:6a:94:12:a6:28) |
| DHCP | Dynamic Host Configuration Protocol | Automatically assigns IP addresses to devices |
| DNS | Domain Name System | Translates names to IP addresses |
| MTU | Maximum Transmission Unit | Largest packet size the interface can carry |
| Duplex | — | Full: send + receive simultaneously. Half: one direction at a time |

### Hardware / switch concepts

| Term | Meaning |
|---|---|
| DSA | Distributed Switch Architecture — modern Linux switch framework in OpenWrt 21.02+ |
| swconfig | Older OpenWrt switch configuration tool used before DSA |
| PHY | Physical layer chip — the Ethernet transceiver silicon |
| SFP | Small Form-factor Pluggable — hot-swappable optical/copper transceiver |
| DDM | Digital Diagnostic Monitoring — SFP feature that reports temp / voltage / power |
| PoE | Power over Ethernet — sends power + data over one cable |
| PSE | Power Sourcing Equipment — the device that supplies PoE |

### OpenWrt ecosystem

| Term | Meaning |
|---|---|
| OpenWrt | Open-source router firmware, Linux-based |
| ImmortalWrt | A fork of OpenWrt maintained by the Chinese community |
| LuCI | OpenWrt's web-based configuration interface |
| UCI | Unified Configuration Interface — OpenWrt's config system |
| opkg | Package manager used by OpenWrt 24.10 and older (`.ipk` files) |
| apk | Newer package manager used by OpenWrt 25.12+ (`.apk` files) |
| rpcd | The service behind LuCI that processes web interface requests |
| uhttpd | The lightweight HTTP server that serves LuCI web pages |
| ubus | OpenWrt's internal message bus used by rpcd, netifd, and LuCI |
| JSHN | JSON shell library used by rpcd backends |
| netifd | OpenWrt's network interface daemon |
| procd | OpenWrt's service manager |
| hotplug | Scripts that run when hardware events happen (e.g. cable plugged in) |
| uci-defaults | Scripts that run once on first install, then delete themselves |

### Port and interface naming

| Name | Meaning |
|---|---|
| eth0, eth1 | Ethernet interfaces — usually physical or CPU-internal |
| lan1, lan2, lan3 | LAN physical ports |
| wan | WAN port — for internet/ISP connection |
| br-lan | LAN bridge — combines multiple ports into one network |
| bat0 | BATMAN-adv virtual mesh interface (not used by this plugin) |
| tailscale0 | Tailscale VPN tunnel interface |
| pppoe-wan | PPPoE WAN tunnel interface |

### Shell commands used in this README

| Command | Meaning |
|---|---|
| uci set | Set a value in UCI config |
| uci show | Display current UCI config |
| ubus call | Invoke an RPC method |
| ubus list | List registered RPC objects |
| opkg install | Install a package with opkg |
| apk add | Install a package with apk |
| ip link show | Show interface status |
| /etc/init.d/x restart | Restart a service |
| logread | Read the system log |
| rm -rf | Recursively delete files (use with care) |
| chmod | Change file permissions |

### Units and measurements

| Unit | Meaning |
|---|---|
| B/s | Bytes per second |
| KB/s | Kilobytes per second = 1024 B/s |
| MB/s | Megabytes per second = 1024 KB/s |
| GB/s | Gigabytes per second = 1024 MB/s |
| Kbit/s | Kilobits per second — used for link speeds |
| Mbit / Gbit | Megabit / Gigabit — 1 Gbit = 1000 Mbit |
| Mb/s | Megabits per second — link speed (different from MB/s) |
| °C | Degrees Celsius — SFP module temperature |
| dBm | Decibel-milliwatts — optical power in SFP modules |
| mA | Milliamperes — bias current in SFP modules |
| V | Volts — SFP supply voltage |

## License

MIT (c) 2026 Arafat Rahman Zami

See [LICENSE](LICENSE) for the full text.
