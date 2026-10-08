# Changelog

## [1.0.13] - 2026-10-05

### Fixed
- Duplicate Port status section from LuCI auto-loader
- JS moved out of web include directory

## [1.0.12] - 2026-10-05

### Fixed
- apk/opkg conflict with luci-mod-status
- uci-defaults swaps JS into place

## [1.0.11] - 2026-10-04

### Added
- swconfig detection
- Single-port fallback

### Fixed
- HTMLSpanElement rendering bug

## [1.0.10] - 2026-10-04

### Added
- Auto-detect DSA vs swconfig

## [1.0.9] - 2026-10-04

### Fixed
- RPC backend list case

## [1.0.8] - 2026-10-04

### Added
- Initial fork of 4IceG/luci-app-ports-status-mod
- Fast cached backend (0.01s)
- Safe uci-defaults cleanup
- ACL file

## [1.0.14] - 2026-10-05

### Added
- **Real-time Rx/Tx speed** under each port card (live update every 3s)
- **Port Details modal** — click the small "i" on any port card to see:
  - Link speed, duplex, operstate, carrier, MTU, MAC
  - SFP module diagnostics (temperature, voltage, TX/RX power)
  - PoE status detection
  - ethtool hardware statistics (when ethtool is installed)
- **Auto-detect architecture** in the extension backend:
  - DSA: reads `/sys/class/net` + `/proc/net/dev`
  - swconfig: parses `swconfig dev switchN show`
  - Single-port: reads the one physical interface
- **Extension backend** (`ports-status-mod-ext`) with new RPC methods:
  `getPortSpeed`, `getSfpInfo`, `getPoeStatus`, `getPortDetails`

### Improved
- **Speed display**: two-line layout, no overflow outside the card, no blank flash during LuCI re-render (uses module-scope cache)
- **Drag-and-drop**: uses pointer events, threshold 6px, follows cursor visually, works from any part of the card (icon, label, description)
- **Drag**: capture deferred until drag begins — clicks on the label still open the edit modal
- **Native image drag** disabled — prevents the browser's default image drag from hijacking card drags

### Fixed
- Port Details modal no longer shows "unknown" — reads params from rpcd stdin JSON
- Speed no longer flickers to `--` every 5 seconds

[1.0.14]: https://github.com/arafatrahmanzami/luci-app-ports-status-mod/compare/v1.0.13...v1.0.14

## [1.0.15] - 2026-10-08

### Fixed
- **Enable / disable LAN ports from the UI now actually toggles the interface** — previously the backend wrote to the state file only, without calling `ip link set`
- `setPortStatus` now reads `port` + `status` from rpcd's stdin JSON (same bug that affected `getPortDetails`)
- Returns structured `{success, port, status}` response for better frontend feedback

[1.0.15]: https://github.com/arafatrahmanzami/luci-app-ports-status-mod/compare/v1.0.14...v1.0.15
