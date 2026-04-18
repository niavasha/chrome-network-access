# Browser Permissions & Network Access Probe

A single self-contained HTML page that lets anyone see, in plain English, what a modern website can learn about them through browser permission and network-access APIs — including the new Chrome **Local Network Access** prompt you've started to see on sites like Amazon and various retailers.

**Live demo:** https://niavasha.github.io/chrome-network-access/

## What it shows

Each card triggers one browser API and reports both the raw data and a human-readable interpretation.

| Probe | Prompt? | What it reveals |
|---|---|---|
| Passive Fingerprint | none | UA, platform, screen, timezone, cores, RAM, language — the silent fingerprint every site reads |
| WebRTC Local IPs | none | Whether your Chrome still leaks real LAN IPs, or correctly hides them behind mDNS |
| Local Network Probe | **LNA** | Fetches common private addresses. On a public origin this triggers Chrome's Local Network Access dialog. Timing reveals live devices on your LAN. |
| Localhost Port Scan | none | Which TCP ports are open on your machine. Any website you visit can map them. |
| Geolocation | yes | GPS-accurate coordinates |
| Notifications | yes | System notification permission |
| Camera & Microphone | yes | Device names (often personally identifying) |
| Clipboard | yes | Contents of your clipboard |
| Web Bluetooth | yes | Nearby BLE devices |
| WebUSB | yes | Vendor, product, serial — enough to uniquely identify hardware |
| Web Serial | yes | Serial port info |
| WebHID | yes | Keyboards, mice, controllers, custom HID |
| Battery | deprecated | Battery level (removed from most browsers due to tracking abuse) |
| Storage Quota | none | Disk quota — a weak fingerprint signal |
| Permissions API | none | State of every permission — silently tells sites what you've already granted |

## Privacy

**Nothing is collected, stored, or transmitted.**

- Runs 100% in your browser.
- No analytics, no cookies, no remote logging, no backend.
- The only requests that leave your machine are the ones you deliberately click (fetches to your own LAN, fetches to localhost, etc.).
- You can audit this yourself — single file, no build step, no dependencies. Open DevTools → Network and watch.

## Run it

**Option A — GitHub Pages (easiest, and the only way to see the real Chrome LNA prompt):**
Visit https://niavasha.github.io/chrome-network-access/

**Option B — Clone locally:**
```bash
git clone https://github.com/niavasha/chrome-network-access.git
cd chrome-network-access
open index.html
```
Note: when loaded via `file://` Chrome won't show the Local Network Access prompt because the origin itself is already considered private. Use GitHub Pages (or serve it from any public HTTPS origin) to see the real dialog.

## Why this exists

Chrome 130+ rolled out **Local Network Access** prompts. Users are seeing this permission appear on ordinary shopping sites (Amazon, retailers running Shopify Plus, etc.) and are rightly asking "what on earth is this site doing with my network?" The answer is almost always third-party anti-fraud or first-party device-intelligence — fingerprinting that survives VPNs and IP changes by probing your LAN.

This page reproduces every such probe so you can see exactly what data each API hands over, and decide for yourself whether to grant or deny.

## License

MIT — see [LICENSE](LICENSE).
