# Home Assistant App: VNC Browser

## Configuration

```yaml
screens:
  - name: Kitchen
    url: "http://homeassistant/lovelace/kitchen"
    resolution: "1280x800"
    depth: 24
    view_only: false
    browser_args: "--force-device-scale-factor=1.25"
  - name: Radar
    url: "https://example.com/radar"
    resolution: "800x480"
vnc_password: ""
log_level: info
```

### Option: `screens`

A list with one entry per screen. Screens are numbered in the order they appear:
the first is screen 1, the second is screen 2, and so on. A screen's number
determines its X display (`:1`, `:2`, …), its VNC port (5901, 5902, …) and its
noVNC port (6901, 6902, …).

Reordering or removing a screen therefore shifts the numbering of the screens
after it, which moves both their ports and their browser profiles. Append rather
than insert if you want existing screens to keep their sessions and addresses.

#### Option: `screens[].url` (required)

The page to open.

To reach Home Assistant itself, use the `homeassistant` hostname rather than a
`.local` address — the app resolves it on the internal Docker network, so it
does not depend on your LAN's DNS and keeps working if the host's address
changes.

**Which port depends on how your Home Assistant was installed.** Since 2026.8 a
*new* Home Assistant OS installation serves on port 80, so no port is needed:

```yaml
url: "http://homeassistant/"
```

Installations set up before 2026.8 keep the port they already had, and Home
Assistant Container installations still default to 8123. For those, ask for it
explicitly:

```yaml
url: "http://homeassistant:8123/"
```

Settings → System → Network shows which port yours is using. If a screen comes
up blank or refuses to connect, this is the first thing to check.

#### Option: `screens[].resolution` (required)

The screen size, as `WIDTHxHEIGHT`, for example `1280x800`. This is the size of
the virtual display and of the Chromium window on it. Viewers scale it to fit
their own window, so it does not have to match the viewing device exactly, but
matching it avoids scaling artefacts on a fixed-size kiosk.

#### Option: `screens[].depth`

Colour depth in bits: `16`, `24` (default) or `32`. Dropping to `16` noticeably
reduces bandwidth, which helps on slow or wireless clients.

#### Option: `screens[].view_only`

When `true`, the screen ignores all keyboard and pointer input. This is enforced
by the VNC server itself, so it applies to every viewer — noVNC and native VNC
clients alike, on every port. Defaults to `false`.

#### Option: `screens[].show_controls`

noVNC draws its own toolbar — a small tab at the left edge that opens a panel
with settings, fullscreen, clipboard and a disconnect button. On a kiosk it is
something for a passer-by to poke at, and it sits oddly next to a browser that
is already running without any UI of its own, so it is **hidden by default** on
a screen's own port.

Set this to `true` to bring it back for a screen — useful if you want the
clipboard, or to send Ctrl+Alt+Del from a tablet.

The Web UI always keeps the toolbar, whatever this is set to; that view is for
you rather than for a kiosk.

Only the toolbar is hidden. Reconnection, scaling and everything else about the
session behave exactly the same.

#### Option: `screens[].browser_args`

Extra command-line arguments for this screen's Chromium, as a single string.
Quoted values are kept intact, so `--user-agent="My Kiosk 1.0"` works.

#### What is already applied

You do not need to ask for any of this — every screen gets it:

`--kiosk --start-fullscreen` means there is **no URL bar, no tab strip, no
toolbar and no window chrome**; the page fills the screen. Alongside that:
`--noerrdialogs`, `--disable-infobars`, `--disable-session-crashed-bubble` (no
"restore pages?" bubble after a power cut), `--no-first-run`,
`--no-default-browser-check`, translation prompts off, and component/version
update checks disabled so a kiosk never surprises you mid-shift.

A managed Chromium policy is also applied, which turns off the things that
interrupt a kiosk with a dialog: the password manager's "save password?"
prompt, address and card autofill, sign-in and sync prompts, notification
requests, translation offers and metrics reporting. Unlike `browser_args` this
is installation-wide, so it applies to every screen.

#### Useful additions

All of these were checked against the Chromium build in this app:

| Argument                                                                                                              | Effect                                                                                                                                |
| --------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| `--force-device-scale-factor=1.5`                                                                                     | Zoom the page; good for small panels                                                                                                  |
| `--hide-scrollbars`                                                                                                   | Hide scrollbars                                                                                                                       |
| `--disable-pinch`                                                                                                     | Stop pinch-zoom changing the layout                                                                                                   |
| `--overscroll-history-navigation=0`                                                                                   | Stop a swipe/drag from navigating back                                                                                                |
| `--disable-notifications`                                                                                             | Suppress web notification prompts                                                                                                     |
| `--autoplay-policy=no-user-gesture-required`                                                                          | Let camera or video streams start on their own                                                                                        |
| `--disable-background-timer-throttling` `--disable-renderer-backgrounding` `--disable-backgrounding-occluded-windows` | Keep a dashboard updating when nothing is watching it. Worth adding for any screen that must stay live                                |
| `--kiosk-printing`                                                                                                    | Print without a print dialog                                                                                                          |
| `--incognito`                                                                                                         | Keep no history or cookies. **Conflicts with persistence** — the screen will be logged out on every restart                           |
| `--force-dark-mode`                                                                                                   | Dark-mode Chromium's own UI. Darkening page content too also needs `--enable-features=WebContentsForceDark`, and results vary by page |

The throttling trio is the one most people end up wanting: without it Chromium
slows timers on a window it thinks nobody is looking at, and a VNC screen with
no viewer attached looks exactly like that.

#### Keeping a screen on one site

Command-line arguments cannot restrict where a screen may navigate. Kiosk mode
removes the browser's own navigation UI, but a link on the page will still take
the screen somewhere else, and nothing brings it back without a restart.

Restricting navigation needs Chromium's enterprise policy (`URLAllowlist` /
`URLBlocklist`), which is read from a file in the image rather than from
command-line arguments — and applies to the whole Chromium installation, so it
would be shared by every screen rather than set per screen. This app does not
currently expose it. If you want it, say so and it can be added as an option.

### Option: `vnc_password`

Optional password for VNC and noVNC connections, enforced by the VNC server
itself. Leave it empty to disable authentication entirely. See
[Passwords and kiosks](#passwords-and-kiosks) for how a keyboard-less device
gets past it.

### Option: `log_level`

One of `trace`, `debug`, `info` (default), `notice`, `warning`, `error` or
`fatal`.

At `info` and above, a small set of known-harmless startup messages is filtered
out of the log: X keymap warnings about duplicate function keys, openbox
noting there is no Xinerama, and Chromium's Google Cloud Messaging and D-Bus
chatter. They are unavoidable in a container, are not signs of a problem, and
otherwise fill the log with things that look like errors.

Set the level to `debug` or `trace` to see every line exactly as the programs
emit it. Only those specific patterns are ever removed — anything unrecognised
always reaches the log.

## Reaching a screen

Every screen has **its own noVNC port** — screen 1 on 6901, screen 2 on 6902,
and so on. A device opens `http://<home-assistant-host>:6901/` in any browser
and lands on that screen. It needs nothing else: no Home Assistant account, no
Ingress, no access to Home Assistant at all.

Each of those servers is started with its target VNC screen fixed on the command
line, and no token routing is loaded. A client on port 6901 cannot ask to be
sent anywhere else — there is no parameter for it. **One port reaches exactly
one screen**, which is what makes the firewall rule below meaningful.

**Only screen 1 is published out of the box** (6901 for noVNC, 5901 for VNC).
Every other screen runs, and is reachable through the Web UI, but has no host
port until you give it one.

To expose another screen, open the app's **Network** settings, find the row for
that screen — they are labelled, for example `Screen 2 - noVNC, open in a
browser` — type a host port into the empty box, save, and restart the app. The
host port does not have to match: you can publish screen 2's noVNC on 8080 if
you prefer.

Leave the rows for screens you are not using empty; an empty box means the port
is not published at all, which is what keeps unused screens off your network.

### Firewalling a device to one screen

The intended setup for untrusted devices — old tablets, e-readers, anything
unpatched — is to give each device access to exactly one port and nothing else:

```
allow  tablet-kitchen  -> ha-host:6901   # screen 1, kitchen dashboard
allow  frame-hallway   -> ha-host:6902   # screen 2, radar
deny   tablet-kitchen  -> ha-host:*      # no Home Assistant, no other screens
deny   frame-hallway   -> ha-host:*
```

A device restricted this way can see and drive its own screen and nothing else.
It never talks to Home Assistant, and it cannot reach another device's screen.

Note that the browsing itself happens in the app, so a screen pointed at your
dashboard is a logged-in session. Anyone who can reach that screen's port can use
that session — the isolation is between screens, not between a device and the
page you told it to display.

### VNC clients

Each screen also has a raw VNC port: 5901 for screen 1, 5902 for screen 2, and
so on. Publish it the same way and point any VNC client at it. The same one
port, one screen property holds.

**macOS Screen Sharing needs `vnc_password` to be set.** With no password the
server offers only the "None" security type, which Apple's built-in client does
not accept — it asks for a password anyway and then hangs. Set `vnc_password`
and connect with that. Other clients (TigerVNC, RealVNC, Remmina) connect
without one. Note that the VNC password format only carries the first 8
characters, so anything longer is silently truncated by client and server
alike.

### From Home Assistant

The app's Web UI shows a screen and, when more than one is configured, a row of
buttons along the top to switch between them. The viewer stays put as you
switch, so there is no going back and forth through a menu, and the last screen
you looked at is remembered. With a single screen the button row is hidden and
the viewer fills the window.

This runs over Ingress, so it is covered by Home Assistant's own authentication
and needs no published ports. It is meant for you, not for the kiosks — it is
the one place that can reach every screen.

## Passwords and kiosks

When `vnc_password` is set, opening a screen's plain URL leaves noVNC to ask for
the password, which is a problem on a device with no keyboard.

Both behaviours are available, chosen by the URL you configure on the device:

| URL                                    | Behaviour                       |
| -------------------------------------- | ------------------------------- |
| `http://ha-host:6901/`                 | Prompts for the password        |
| `http://ha-host:6901/?password=SECRET` | Connects immediately, no prompt |

Give the keyboard-less kiosk the second form and it will come up on its own
after a power cut. The password is not discoverable from the plain URL — nothing
the server hands out contains it — so it stays a secret you deploy to a device,
not something a visitor to the port can read.

Any other noVNC setting can be appended the same way, for example
`?password=SECRET&resize=off` to stop scaling, or `&quality=3` on a slow link.

The Web UI supplies the password for you, since it already required a Home
Assistant login.

## Limits

**Only screen 1 is published by default.** Screens 2-8 need a host port set
under Network settings before anything outside Home Assistant can reach them.

**Ports 5901-5908 and 6901-6908 are declared, so screens 1-8 can be published.**
Home Assistant silently discards port mappings for ports an app does not
declare, so this is a hard ceiling that cannot be lifted from the Network panel
or the CLI. A ninth screen still works — through the Web UI, and to other
apps on the internal network — but cannot be published to your LAN. Raising
the ceiling means editing `ports` and `ports_description` in this app's
`config.yaml` and installing it from your own copy of the repository.

**The noVNC ports are plain HTTP.** Traffic between a device and the app is
unencrypted, and so is a password sent in the URL. This suits the intended
setup — old devices on a LAN, restricted by firewall — but do not route these
ports over an untrusted network or the public internet.

## How it fits together

```
    old tablet ─────────────► :6901 ─── noVNC (fixed target) ──┐
    e-ink frame ────────────► :6902 ─── noVNC (fixed target) ─┐│
    VNC client ─────────────► :5901 ──────────────────────────┐││
                                                              │││
    Home Assistant Ingress ── noVNC (all screens, by token) ─┐ │││
                                                             │ │││
                         screen 1        screen 2        screen 3
                         Xvnc :1         Xvnc :2         Xvnc :3
                         openbox         openbox         openbox
                         chromium        chromium        chromium
```

`Xvnc` is both the X server and the VNC server, so a screen is one X display,
one window manager, one browser and one VNC endpoint, plus its own noVNC server.

Only the Ingress server can reach every screen, and it is never published. Each
per-screen server is pinned to its own screen.

Every process is supervised individually: if a browser or an X server dies, only
that screen's processes restart, and the other screens keep running.

## Migrating from ha-vnc-web-browser

The options are deliberately close, with two differences:

- `displays` is called `screens`.
- There is no `port` field. A screen's ports come from its position in the list,
  so put your displays in ascending port order and drop the field.
