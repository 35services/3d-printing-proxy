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

How the fork describes itself ([README](https://github.com/fmaass/teedy-docs#readme)):

> This is an actively maintained fork of sismics/docs (Teedy). It includes
> OIDC/SSO authentication, Java 21 + Jetty 12 modernization, security
> hardening, and multi-arch Docker images published to GHCR.

(The image deployed here, v3.8.10, actually runs Java 25: `openjdk 25.0.4`.)

Upstream [sismics/docs](https://github.com/sismics/docs) does not publish
arm64 images, so the Pi runs the fork
[fmaass/teedy-docs](https://github.com/fmaass/teedy-docs) instead. It ships
multi-arch images (amd64 + arm64) and adds OIDC, Java 25 and Jetty 12. Its
version numbers (3.8.x) are its own and don't match upstream's (1.12 as of
2026-11-23).

### Deployed version

There are several Teedy lines around, so name the fork, not just the number:

| | Repo | Versions | Deployed here |
|---|---|---|---|
| Upstream | [sismics/docs](https://github.com/sismics/docs) | 1.x (1.12 on 2026-11-23) | no, no arm64 images |
| Fork | [fmaass/teedy-docs](https://github.com/fmaass/teedy-docs) | 3.x | **yes** |

Deployed now: **fmaass/teedy-docs `v3.8.10`**, commit `95d4187`, image
`ghcr.io/fmaass/teedy-docs:v3.8.10`.

The landing-page card shows the same line. Over HTTPS it reads the version
and commit from the running instance (`/teedy/api/app`, fields
`current_version` and `commit_id`); on the plain-HTTP LAN address it shows the
version written into `site/index.html`. To check by hand:

    curl -s https://<TS_DOMAIN>/teedy/api/app        # current_version, commit_id
    docker inspect teedy --format '{{.Config.Image}}' # on the Pi

`commit_id` is a commit in the fork, so it tells the builds apart even if
another fork uses the same version number.

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
- [x] Check that the fork has upstream's recent security fixes (checked
      2026-11-24 against the v3.8.10 source):
  - sismics/docs #798 and #810 (stored XSS via HTML, SVG or XML uploads):
    covered, more strictly. Every uploaded original is served as an
    attachment with `Content-Security-Policy: sandbox; default-src 'none'`
    and `nosniff`, whatever its MIME type.
  - #799 (LDAP injection): covered, the username is escaped per RFC 4515.
  - #806 (rename/delete a file with read access only): covered, both need
    write permission.
  - #810 (HttpOnly session cookie): covered, `auth_token` is HttpOnly and
    SameSite=Lax, with a separate CSRF token.
- [ ] Set the default language to German in the admin settings: the app
      reports `eng` although `.env` sets `DOCS_DEFAULT_LANGUAGE=deu`.
- [ ] Remove `~/teedy-admin-password.txt` from the Pi once the admin password
      has been changed in the UI.
- [ ] Decide who gets accounts and whether to turn on OIDC.

### Updating

Read the fork's changelog, set `TEEDY_VERSION` in `.env` to the new tag, then
`docker compose pull && docker compose up -d` in `~/Documents/teedy`.
Then update the version in [Deployed version](#deployed-version) and the
hard-coded fallback on the Teedy card in `site/index.html`.

### Removing it

`docker compose down` in `~/Documents/teedy`, delete the `/teedy` block from
the `Caddyfile` and the Teedy card from `site/index.html`, and delete
`~/Documents/teedy/data` once nothing in it is needed.
