# Changelog

All notable changes to Netatmo Plus are documented here.

---

## [Unreleased]

### Added — Climate
- `open_window` attribute — whether open-window detection has suppressed heating
- `anticipating` attribute — whether the thermostat is pre-heating for the next scheduled slot
- `setpoint_end_time` attribute — ISO datetime when the current manual override expires (null if in schedule mode)
- `away_temperature` attribute — configured away-mode setpoint from the active schedule
- `frost_guard_temperature` attribute — configured frost-guard setpoint from the active schedule
- `heating_power_request` attribute — now exposed for NATherm1 rooms (was NRV valves only)

### Added — Camera
- `reachable` attribute — actual device reachability (was not exposed, availability was incorrectly derived from `alim_status`)
- `wifi_strength` attribute — Wi-Fi signal strength (integer)
- `firmware` attribute — human-readable firmware version string (e.g. `NOC-3.3.1`)

### Fixed — Camera
- `light_state` — was only updated via webhook (null after restart). Now initialized from the polled `device.floodlight` value on each update cycle, with webhook updates still applied on top.

### Added — Siren
- Automatic re-login on token expiry — when the Netatmo web session token expires (code 3), the integration automatically re-logs in and retries the command transparently. No user intervention required.

---

## [v1.2.0] — 2026-04-15

### Added — Climate
- `scheduled_temperature` attribute — the scheduled setpoint temperature for the room at the current time, computed from the active Netatmo schedule timetable
- `scheduled_zone_name` attribute — the name of the currently active schedule zone (e.g. Confort, Eco, Nuit) as defined by the user in the Netatmo app

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
