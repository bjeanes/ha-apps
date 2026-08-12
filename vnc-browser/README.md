# Home Assistant App: VNC Browser

Run any number of kiosk browser screens, each with its own URL, resolution and
Chromium flags, and give each one its own address that any browser can open.

![Supports aarch64 Architecture][aarch64-shield]
![Supports amd64 Architecture][amd64-shield]

## What it does

Each configured screen gets its own X display, its own Chromium instance in
kiosk mode, its own VNC server, and **its own noVNC port**. A device opens
`http://<ha-host>:6901/` and lands on screen 1 — no Home Assistant account, no
Ingress, no access to Home Assistant at all.

Each per-screen noVNC server has its VNC target fixed at startup and no token
routing, so a client on one screen's port cannot ask to be sent to another. One
port reaches exactly one screen, which means an untrusted device can be
firewalled to precisely the screen it is meant to show:

```
allow  tablet-kitchen -> ha-host:6901    # its screen, and nothing else
deny   tablet-kitchen -> ha-host:*
```

That is the point: old tablets and e-readers make good dashboards but poor
network citizens. They only ever have to render a VNC stream, while the actual
browsing happens on your Home Assistant host.

## Why both

It combines the two halves of the prior art it is based on:

- [gnyman/havnc][havnc] serves the display over noVNC, so clients only need a
  browser — but it drives a single screen.
- [MindFreeze/ha-vnc-web-browser][ha-vnc-web-browser] drives several
  independent displays, each with its own URL, resolution and Chromium flags —
  but each one is reachable only over raw VNC.

This app does both: many independent screens, each browser-reachable on its
own port.

## Installation

1. Add this repository to your Home Assistant app store.
2. Install the "VNC Browser" app.
3. Configure your screens (see [DOCS.md](DOCS.md)).
4. Start the app. Screen 1 is published on 6901 (noVNC) and 5901 (VNC) already.
5. For screens 2-8, set a host port under the app's Network settings — they
   start unpublished so unused screens stay off your network.

## Configuration

```yaml
screens:
  - name: Kitchen
    url: "http://homeassistant/lovelace/kitchen"
    resolution: "1280x800"
  - name: Radar
    url: "https://example.com/radar"
    resolution: "800x480"
    view_only: true
```

| Screen | noVNC (browser) | VNC client |
| ------ | --------------- | ---------- |
| 1      | 6901            | 5901       |
| 2      | 6902            | 5902       |
| …      | 690n            | 590n       |

Screens 1-8 can be published; further screens are reachable through the Web UI
only. Full option reference and the firewall/password guidance are in
[DOCS.md](DOCS.md).

[aarch64-shield]: https://img.shields.io/badge/aarch64-yes-green.svg
[amd64-shield]: https://img.shields.io/badge/amd64-yes-green.svg
[havnc]: https://github.com/gnyman/havnc
[ha-vnc-web-browser]: https://github.com/MindFreeze/ha-vnc-web-browser
