# Changelog

## 0.1.0

Initial release.

**Screens**

- Any number of screens, each with its own URL, resolution, colour depth and
  Chromium arguments.
- Per-screen view-only mode, enforced by the VNC server so it applies to every
  client on every port.
- Per-screen browser profiles under `/data`: logins survive restarts, and one
  screen's session is separate from the next.

**Access**

- A dedicated noVNC port per screen (6901+), each pinned to a single screen
  with no token routing, so a device can be firewalled to exactly one screen
  and needs no Home Assistant access at all.
- A raw VNC port per screen (5901+) for native VNC clients.
- Screen 1 is published by default; screens 2-8 need a host port set under
  Network settings. Ports are declared for screens 1-8; further screens stay
  reachable through the Web UI and the internal network.
- Optional VNC password, with a `?password=` URL form so a keyboard-less kiosk
  can connect unattended.
- A Web UI over Ingress, covered by Home Assistant's own authentication, with a
  switcher for moving between screens.

**Kiosk behaviour**

- Chromium runs with no URL bar, tab strip or window chrome, and a managed
  policy suppresses the dialogs that interrupt a kiosk: "save password?",
  autofill, sign-in and sync prompts, notification requests, translation offers
  and metrics reporting.
- noVNC's own toolbar is hidden on a screen's own port, so a kiosk shows
  nothing but the page. `show_controls: true` brings it back per screen.
- Known-harmless startup messages are kept out of the log; `debug` and `trace`
  show the raw output.
