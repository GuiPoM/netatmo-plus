# Changelog

All notable changes to Netatmo Plus are documented here.

---

## [v1.4.2] — 2026-05-07

### Fixed
- Siren credentials lost after saving options — `web_auth` was not updated when options changed (only initialized at startup)

---

## [v1.4.1] — 2026-05-06

### Fixed
- Siren entity no longer created after update to 2026.5.0 — `NETATMO_CREATE_CAMERA_SIREN` signal was missing from `data_handler.py` camera category dispatch list

---

## [v1.4.0] — 2026-05-06

### Changed
- Based on Home Assistant 2026.5.0 Netatmo integration (updated from 2026.3.0)

---

## [v1.3.6] — 2026-04-18

### Fixed — Camera
- `monitoring` attribute: now correctly reflects webhook on/off updates while falling back to polled API value at startup (fixes `monitoring: null` on first load without breaking webhook-based updates)

### Fixed — Siren
- Siren entity availability now based on `alim_status` instead of webhook status — siren is available as long as the camera is powered, regardless of webhook registration state

### Fixed — Camera
- `reachable` attribute: show `None` when not initialized instead of misleading `false`

---

## [v1.3.3] — 2026-04-16

### Fixed
- Missing `callback` import in `__init__.py` causing startup error

---

## [v1.3.2] — 2026-04-16

### Fixed
- Web session auth now updated immediately when siren credentials are saved via options flow — no more manual reload required

---

## [v1.3.1] — 2026-04-16

### Fixed
- Missing `CONF_SIREN_EMAIL` and `CONF_SIREN_PASSWORD` imports in `__init__.py` causing startup error

---

## [v1.3.0] — 2026-04-16

### Added — Siren
- Automatic re-login on token expiry — transparent, no user intervention required
- Password stored in options for automatic re-authentication

### Added — Climate
- `open_window` attribute — open window detection active
- `anticipating` attribute — pre-heating for next scheduled slot
- `setpoint_end_time` attribute — ISO datetime when manual override expires
- `away_temperature` attribute — configured away-mode setpoint from active schedule
- `frost_guard_temperature` attribute — configured frost-guard setpoint
- `heating_power_request` attribute — now exposed for NATherm1 rooms (was NRV valves only)

### Added — Camera
- `reachable` attribute — device reachability
- `wifi_strength` attribute — Wi-Fi signal strength (integer)
- `firmware` attribute — human-readable firmware version string

### Fixed — Camera
- `light_state` — was only updated via webhook (null after restart). Now initialized from polled `device.floodlight` value

---

## [v1.2.0] — 2026-04-15

### Added — Climate
- `scheduled_temperature` attribute — scheduled setpoint temperature for the room at current time
- `scheduled_zone_name` attribute — name of the currently active schedule zone (e.g. Confort, Eco, Nuit)

---

## [v1.1.0] — 2026-04-15

### Added
- `scheduled_temperature` and `scheduled_zone_name` attributes on climate entities (initial implementation, refined in v1.2.0)

---

## [v1.0.0] — 2026-04-15

### Added
- **Siren control** for NOC (Smart Outdoor Camera) — `siren` entity exposing `turn_on` / `turn_off` services. Uses Netatmo web session authentication (configured via Settings > Devices & Services > Netatmo Plus > Configure). One-time login required, token persists server-side.

### Fixed
- **`monitoring: null`** — `camera.garage` (and any NOC camera entity) now correctly reports `monitoring: true/false` instead of `null`. The official integration was reading an internal webhook-only variable instead of the polled API value.

### Base
- Based on Home Assistant 2026.3.0 Netatmo integration
- All official features (thermostats, weather station, cameras, covers, etc.) preserved
