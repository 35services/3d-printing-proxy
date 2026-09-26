# Changelog

All notable changes to this project are documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). This
Caddy setup runs continuously on one Raspberry Pi rather than being cut into
versioned releases, so entries are grouped by date (when the change was
merged to `main`, which is also when it reaches the Pi) instead of a version
number.

## 2026-09-26

### Added

- Hold a [Wake Lock](https://developer.mozilla.org/en-US/docs/Web/API/Screen_Wake_Lock_API)
  while a webcam preview is enlarged, so the screen doesn't sleep. Only takes
  effect over HTTPS (the plain-HTTP LAN address is not a secure context) and
  isn't supported by Firefox; both cases just no-op instead of erroring.

## 2026-09-25

### Added

- "Stromverbrauch" card on the landing page: live power (W) plus today's,
  yesterday's and total energy (kWh) for the MK3, MK4, A1 mini and Trockner
  Tasmota plugs, polled every 10s.
- Tap a plug row to open a dialog and switch it on/off. The Caddyfile exposes
  only a fixed set of routes per plug (status, relay state, on, off), each
  rewritten to one specific Tasmota command, so no other command can reach
  them.
- Tap a webcam preview to enlarge it to a centred overlay (tap again, or Esc,
  to shrink).

### Changed

- Webcam preview tiles: 4:3 -> 16:9, to match the OctoPrint project's tuned
  stream resolutions.

## 2026-09-22

### Added

- `/sensor-visualizer/` route, reverse-proxied to the sensor log visualizer's
  own Docker container on `127.0.0.1:8000`, with a link from the landing
  page.

## 2026-09-21

### Added

- `/octoprint-mk4/` route (-> `:92`), with its own landing-page card and live
  webcam preview (the preview script now handles multiple simultaneous
  streams).
- HTTPS for the Tailscale `*.ts.net` name: Caddy fetches the certificate
  straight from the local `tailscaled`, no ACME/DNS setup and no CA to
  install on devices. Plain HTTP to that name redirects to HTTPS.
- Vaultwarden served at `/vaultwarden/`, HTTPS only (see the `bitwarden`
  project).

## 2026-09-20

### Added

- Initial Caddy reverse proxy: `/octoprint-mk3/` (-> `:91`) and `/filaments/`
  (-> `:81`) on port 80, with a static landing page showing a live MK3 webcam
  preview.
- Landing-page footer linking the source repos behind the setup.

### Fixed

- Landing page's "OctoApp" note, which initially told people *not* to use
  the proxy URL - corrected to recommend it.
