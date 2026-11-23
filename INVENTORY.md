# Inventory

What runs on the Pi (`octo35services`, Raspberry Pi 4 Model B, arm64) and
where it lives. Each service is its own Docker Compose project under
`~/Documents/<project>`; this repo is `~/Documents/caddy`. Routes are in the
[README](README.md#routing).

| Service | Project | Image | Port on the Pi | Status |
|---|---|---|---|---|
| Caddy proxy + landing page | `caddy` (this repo) | `caddy:latest` | `:80`, `:443` | production |
| OctoPrint (two printers) | `octoprint` | `octoprint-prusaslicer:latest` (local build) | `:91`, `:92` | production |
| Filament tool | `simple-filament-tool` | `simple-filament-tool-app` (local build) | `:81` | production |
| Sensor visualizer | (separate project) | `sensor-visualizer-visualizer` (local build) | `127.0.0.1:8000` | production |
| Vaultwarden | `bitwarden` | `vaultwarden/server:latest` | `127.0.0.1:8080` | production |
| Portainer | `portainer` | `portainer/portainer-ce:latest` | `:9443` | production |
| Laptop checker | `das-macht-schule` | `das-macht-schule-laptop-checker` (local build) | `:9000` | production |
| **Teedy** + PostgreSQL | `teedy` | `ghcr.io/fmaass/teedy-docs:v3.8.10`, `postgres:17-alpine` | `127.0.0.1:8090` | **experiment** |

## Teedy prototype (experiment)

[Teedy](https://teedy.io) is a document management system: upload PDFs and
images, tag them, and find them again by full-text search, with OCR on
scanned pages. The idea is to keep manuals, datasheets and invoices for the
workshop in one searchable place.

**Status: experiment, set up 2026-11-22.** It is linked from the landing page
with an "Experiment" badge. Nothing in it should be the only copy of a
document yet, and it may be removed again.

### Why the fmaass fork

Upstream [sismics/docs](https://github.com/sismics/docs) does not publish
arm64 images, so the Pi runs the fork
[fmaass/teedy-docs](https://github.com/fmaass/teedy-docs) instead. It ships
multi-arch images (amd64 + arm64) and adds OIDC, Java 25 and Jetty 12. Its
version numbers (3.8.x) are its own and don't match upstream's (1.12 as of
2026-11-23).

### Setup

- Compose project `~/Documents/teedy` (not in git): `teedy` (the app) and
  `teedy-db` (PostgreSQL 17). Secrets and `TEEDY_VERSION` are in `.env`.
- Only listens on `127.0.0.1:8090`; Caddy serves it at
  `https://<TS_DOMAIN>/teedy/`, so it is reachable over Tailscale HTTPS only.
- The Java heap is capped at 1 GB (`-Xmx1g`) to leave room for the other
  services.
- Data in `~/Documents/teedy/data`: `teedy/` holds the stored files
  (encrypted) and the search index, `postgres/` the database. The two are only
  usable together.

### State (2026-11-24)

- Running and healthy on v3.8.10, the fork's latest release (2026-09-02).
- No documents yet, 2 users. OCR on, guest login and OIDC off.

### Before it stops being an experiment

- [ ] Nightly backup of the database and `data/teedy` together
      (`pg_dump` + `tar`, see the README in the `teedy` project).
- [ ] Check that the fork has upstream's recent security fixes
      (sismics/docs #798 stored XSS via HTML uploads, #799 LDAP injection,
      #806 file rename/delete permissions, #810 XML content types, CSP,
      HttpOnly cookie).
- [ ] Set the default language to German in the admin settings: the app
      reports `eng` although `.env` sets `DOCS_DEFAULT_LANGUAGE=deu`.
- [ ] Remove `~/teedy-admin-password.txt` from the Pi once the admin password
      has been changed in the UI.
- [ ] Decide who gets accounts and whether to turn on OIDC.

### Updating

Read the fork's changelog, set `TEEDY_VERSION` in `.env` to the new tag, then
`docker compose pull && docker compose up -d` in `~/Documents/teedy`.

### Removing it

`docker compose down` in `~/Documents/teedy`, delete the `/teedy` block from
the `Caddyfile` and the Teedy card from `site/index.html`, and delete
`~/Documents/teedy/data` once nothing in it is needed.
