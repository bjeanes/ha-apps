# Home Assistant App: pgweb

## How to use

1. Install the app
2. Set the `db` option to the database you want to browse
3. Start the app
4. Press "Open Web UI"
5. Optionally, add it to the sidebar for easier access

## Configuration

### Option: `db` (required)

A PostgreSQL connection URL, for example:

```text
postgresql://postgres:homeassistant@77b2833f-timescaledb/homeassistant?sslmode=disable
```

Both `postgresql://` and `postgres://` schemes are accepted.

To reach another app's database, use that app's hostname on the internal
network — the hash-prefixed name shown in its documentation, such as
`77b2833f-timescaledb` — rather than a `.local` address.

Add `?sslmode=disable` when connecting to a database that does not offer TLS,
which is usually the case for a sibling app on the internal network.
