## 0.2.0

- Update pgweb to 0.17.0, from 0.11.11 (2022). See the
  [upstream releases](https://github.com/sosedoff/pgweb/releases) for what
  changed in between. The options this app uses are unaffected.
- Update to base image 21.0.1 (Alpine 3.24, from Alpine 3.15) and move to the
  s6-rc service format it uses. s6-overlay v2 copied service scripts into
  `/var/run/s6` and fixed their permissions on the way; v3 runs them in place
  and does not, so the scripts now carry their own executable bit.
- Publish a single multi-arch image (`ghcr.io/bjeanes/ha-app-pgweb`) instead of
  one image per architecture.
- Drop i386: the current Home Assistant build tooling supports only amd64 and
  aarch64.
- Document the `db` option.

## 0.1.0

- Initial release
