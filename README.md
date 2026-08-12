# bjeanes' Home Assistant app repository

App documentation: <https://developers.home-assistant.io/docs/apps>

> Home Assistant renamed "add-ons" to "apps". This repository followed suit: it
> was previously `bjeanes/hassio-addons`, and everything here that used to be
> called an add-on is now called an app. GitHub redirects the old URL, so a
> Home Assistant instance that added the repository under its former name keeps
> working.

[![Open your Home Assistant instance and show the add app repository dialog with a specific repository URL pre-filled.](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Fbjeanes%2Fha-apps)

## Apps

This repository contains the following apps.

### [pgweb](./pgweb)

Web-based PostgreSQL database browser.

![Supports aarch64 Architecture][aarch64-shield]
![Supports amd64 Architecture][amd64-shield]

## Development

Open this repository in the devcontainer (VS Code will offer to). It runs a
real Supervisor with this repository mounted as local apps, so you can install
and iterate on an app without publishing anything.

The tasks in `.vscode/tasks.json` drive the loop:

1. **Start Home Assistant** — boots Supervisor and Home Assistant
2. **Install App** — installs the local copy of an app
3. **Rebuild and Start App** — rebuilds from your working tree and tails the log

Home Assistant is then on <http://localhost:7123>.

### The `image` key

Supervisor only builds an app from source when its `config.yaml` has **no**
`image:` key; when the key is present it pulls the published image instead. So
while developing an app locally, comment `image:` out, or your changes will
appear to do nothing.

Uncomment it again before committing. CI fails if an app on `main` is missing
its `image:` key, so a commented-out line cannot ship by accident.

### Releasing

Bump the app's `version:` in its `config.yaml` and add a `CHANGELOG.md` entry.
Merging to `main` builds and publishes the image, and Home Assistant offers the
new version once that build finishes.

**The first publish of a new image name needs one manual step.** GitHub creates
the package as private, and Home Assistant pulls anonymously, so the update
fails until you make it public: GitHub → Packages → the package → Package
settings → Change visibility → Public (or link it to this repository and
inherit its visibility). This applies once per image name, not once per
release.

[aarch64-shield]: https://img.shields.io/badge/aarch64-yes-green.svg
[amd64-shield]: https://img.shields.io/badge/amd64-yes-green.svg
