# Browser Permissions & Network Access Probe

> **Understand your browser. Make informed permission choices.**
>
> Modern browsers expose a rich set of APIs — geolocation, camera, Bluetooth, USB, and the new Chrome *Local Network Access* prompt — and each click of "Allow" shares something specific with the site. Most of the time the site has a legitimate need; sometimes it doesn't. Either way, the choice is easier when you know exactly what the data looks like.

A transparency tool that runs each permission and network-access API against your own browser and explains, in plain English, what it actually reveals. The raw JSON is underneath each explanation if you want to see it for yourself.

**The good news:** modern Chrome defends you in depth. Even when you grant Local Network Access, sites still can't read responses from devices that don't opt in with CORS headers. Real local IPs are hidden behind mDNS hostnames by default. This tool helps you confirm those defenses are working — and decide confidently what to allow.

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

## Why Local Network Access is a good thing

Chrome 130+ rolled out **Local Network Access** prompts on ordinary shopping sites (Amazon, retailers running Shopify Plus, etc.). Users are rightly asking "what is this site doing with my network?" The honest answer is that LNA is one of the biggest browser-security wins of the decade — and the prompt exists because Chrome is finally surfacing something that used to happen invisibly.

**Before LNA**, any website you visited could silently send HTTP requests to your router (`192.168.1.1`), printers, NAS, smart TVs, and `localhost` — no prompt, no log. Attackers abused this for years:

- **Router CSRF** — malicious ads could POST to your router's admin panel and change your DNS, redirecting all future traffic through an attacker.
- **IoT exploitation** — drive-by attacks against unpatched cameras, printers, and smart-home hubs reachable only on your LAN.
- **Silent LAN fingerprinting** — ad-tech and fraud vendors mapping your network shape without consent.

**LNA closes these attacks with defence in depth:**

1. The site has to ask permission.
2. Even after you Allow, the target device must opt in via a CORS header to expose its responses.
3. Real local IPs are hidden behind mDNS hostnames (since Chrome 79) so WebRTC can't leak them passively.

This page reproduces each probe so you can see exactly what gets exposed — and confirm the defences are working. Grant or deny with confidence.

## License

MIT — see [LICENSE](LICENSE).
