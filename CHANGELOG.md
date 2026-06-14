# Changelog

All notable changes to Netatmo Plus are documented here.

---

## [v2026.6.0.2] — 2026-06-04

### Fixed
- Re-aligned `camera.py` and `climate.py` with upstream HA 2026.6.0 — entity
  unique_ids now use `device_type_to_str` (`...-DeviceType.NOC`) as in the
  official integration.

> ⚠️ **If you installed v2026.6.0 or v2026.6.0.1**, your camera (NOC, NACamera,
> NDB, NPC) and thermostat/valve (NATherm1, NRV, OTM, NAC, BNS) entities may
> have been duplicated due to an error in the upgrade process. Please delete the
> orphaned entities and rename the new ones to restore their original `entity_id`
> if needed.

---

## [v2026.6.0.1] — 2026-06-04

### Fixed
- Restored old `unique_id` format for camera and climate entities (`...-NOC` instead of `...-DeviceType.NOC`) — the upstream HA 2026.6.0 change broke entity registry and all automations referencing these entities
- Restored `alim_status` availability fix for siren and light entities (regressed during 2026.6.0 sync)

---

## [v2026.6.0] — 2026-06-04

### Changed
- Based on Home Assistant 2026.6.0 core Netatmo integration

### Upstream changes included (2026.6.0 vs 2026.5.0)
- `select.py` — Fix AttributeError when webhook schedule_id is not in cache (HA core #171914)
- `camera.py`, `const.py`, `webhook.py` — Replace duplicate constants with `homeassistant.const` imports (HA core #171953)
- `helper.py` — Fix dataclass default (RUF009: `uuid4()` → `field(default_factory=uuid4)`) (HA core #172738)

---

## [v1.4.4] — 2026-05-28

### Fixed — Siren & Light (NOC camera)

Both `siren` and `light` (floodlight) entities on the NOC camera suffered from
the same conceptual bug: their `available` property was tied to
`data_handler.webhook`, which is `False` at startup and only becomes `True`
after Netatmo's servers send a webhook activation ping.

**Root cause (historical):** when the `available` property was introduced in
January 2021 (HA core #42791), `async_update_callback` relied exclusively on
webhook events for state updates — so gating availability on the webhook was
intentional and correct. During the pyatmo 7.0.1 refactor, `async_update_callback`
was rewritten to read `device.floodlight` from the polled `homestatus` API, but
the `available` property was never updated to match. The two drifted apart silently.

**Fix:** availability is now based on `device.alim_status is not None` (camera
power state, from the polled API) — consistent with `camera.py` and `siren.py`.
The entity is available as soon as the camera is reachable, regardless of webhook
registration state.

This bug also exists in the upstream Home Assistant core Netatmo integration and
will be reported there separately.

---

## [v1.4.4b1] — 2026-05-28 *(beta)*

### Fixed
- Siren entity unavailable after reboot — availability was incorrectly tied to webhook connection status instead of camera power state (`alim_status`). The webhook flag is `False` at startup and only becomes `True` after Netatmo's servers send a confirmation ping, causing the siren to appear permanently unavailable. The entity is now available as soon as the camera is reachable, consistent with the camera entity behaviour.

---

## [v1.4.3] — 2026-05-07

### Fixed
- Siren commands failing with "credentials not configured" — ``_get_web_auth()`` was reading from legacy ``hass.data`` pattern instead of ``data_handler.web_auth`` (HA 2026.5.0 migration to ``runtime_data``)

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
