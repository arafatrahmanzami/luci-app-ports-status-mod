# luci-app-ports-status-mod

Enhanced LuCI port status module for OpenWrt — custom labels, drag-and-drop ordering, live status dots, auto-detect DSA / swconfig / single-port.

## Features

- Custom labels — click any port card to rename
- Descriptions — add a note under each port
- Drag-and-drop reordering
- Live dots: Active (blinking) / Idle / Down / Disabled
- Enable / Disable LAN ports from UI
- Backup / Restore config as JSON
- Persistent at /etc/user_defined_ports.json
- Fast 0.01s cached backend

## Install (OpenWrt 24.10 / IPK)

    opkg install luci-app-ports-status-mod_*.ipk

## Install (OpenWrt 25.12 / APK)

    apk add --allow-untrusted luci-app-ports-status-mod-*.apk

## After Install

    rm -rf /tmp/luci-*
    /etc/init.d/uhttpd restart

Then hard-refresh the browser.

## License

MIT (c) 2026 Arafat Rahman Zami
