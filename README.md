# Netatmo Plus

[![hacs_badge](https://img.shields.io/badge/HACS-Custom-orange.svg)](https://github.com/custom-components/hacs)
[![GitHub release](https://img.shields.io/github/release/GuiPoM/netatmo-plus.svg)](https://github.com/GuiPoM/netatmo-plus/releases)

**A fork of the [official Home Assistant Netatmo integration](https://www.home-assistant.io/integrations/netatmo) with additional features pending upstream merge.**

This is a complete replacement for the built-in integration, adding features that are not yet available in the official integration.

**📖 [View Full Documentation](DEVELOPMENT.md)** — Complete guide for development and maintenance.
**📋 [Changelog](CHANGELOG.md)** — Full release history.

---

## Added Features

### 1. Siren Control for NOC (Smart Outdoor Camera)

**Trigger and stop the built-in siren of the Netatmo Outdoor Camera (NOC/Presence) from Home Assistant.**

The Netatmo public OAuth2 API does not expose siren control — it is only accessible via the Netatmo web session API. This integration handles the authentication transparently.

#### Configuration

After installation:

1. Go to **Settings > Devices & Services > Netatmo Plus**
2. Click the **three dots > Configure**
3. Enter your Netatmo **email** and **password**
4. Click **Submit** — a confirmation email will be sent by Netatmo
5. The siren entity `siren.garage` (or similar) is now operational

**Notes:**
- Netatmo sends a login notification email on first setup
- The session token persists server-side — re-login is automatic on expiry, no user intervention needed

#### Usage

```yaml
# Trigger siren in an automation
action: siren.turn_on
target:
  entity_id: siren.garage

# Stop siren
action: siren.turn_off
target:
  entity_id: siren.garage
```

---

### 2. Additional Climate Attributes

The following attributes are added to thermostat/valve climate entities:

| Attribute | Description |
|---|---|
| `scheduled_temperature` | Current scheduled setpoint from active schedule |
| `scheduled_zone_name` | Active schedule zone name (e.g. Confort, Eco, Nuit) |
| `away_temperature` | Configured away-mode setpoint |
| `frost_guard_temperature` | Configured frost-guard setpoint |
| `open_window` | Whether open-window detection has suppressed heating |
| `anticipating` | Whether the thermostat is pre-heating for the next slot |
| `setpoint_end_time` | ISO datetime when the current manual override expires |
| `heating_power_request` | Heating demand % (NRV valves and NATherm1 rooms) |

---

### 3. Additional Camera Attributes

The following attributes are added to `camera.garage` (and any NOC camera entity):

| Attribute | Description |
|---|---|
| `monitoring` | Fixed: correctly reports `true`/`false` (was always `null`) |
| `light_state` | Fixed: now reflects polled API value (was stale after restart) |
| `wifi_strength` | Wi-Fi signal strength (integer) |
| `reachable` | Device reachability |
| `firmware` | Human-readable firmware version |

---

## Why This Fork Exists

These features are not yet in the official integration because:

1. **Siren control** requires the Netatmo web session API — the public OAuth2 API does not expose `siren_status`. A PR has been submitted to pyatmo ([#554](https://github.com/jabesq-org/pyatmo/pull/554)).
2. **Attribute additions and fixes** are pending upstream contribution.

---

## Installation

### Via HACS (Recommended)

1. Open HACS in your Home Assistant
2. Click on **Integrations**
3. Click the three dots in the top right corner
4. Select **Custom repositories**
5. Add: `https://github.com/GuiPoM/netatmo-plus`
6. Select category: **Integration**
7. Click **Add**
8. Search for **Netatmo Plus**
9. Click **Download**
10. Restart Home Assistant

### Manual Installation

1. Download the latest release from [Releases](https://github.com/GuiPoM/netatmo-plus/releases)
2. Extract the `netatmo` folder from the zip
3. Copy it to your `<config>/custom_components/` directory
4. Restart Home Assistant

---

## Setup

This custom component **replaces** the built-in Netatmo integration. When installed in `custom_components/netatmo/`:

- ✅ Home Assistant will load this version instead of the built-in one
- ✅ All your existing Netatmo configuration continues to work
- ✅ No migration needed — just adds new optional features
- ✅ You can revert anytime by removing the custom component folder and restarting

**Configure as usual** — the integration uses the same OAuth2 flow as the official integration. After setup, configure siren credentials via the **Configure** button.

---

## Compatibility

- **Home Assistant**: 2026.3.0 or later
- Based on Home Assistant 2026.3.0 core Netatmo integration
- Compatible with all official Netatmo features

---

## How to Revert to the Official Integration

```bash
# Remove the custom component folder
rm -rf <config>/custom_components/netatmo/

# Restart Home Assistant
```

Home Assistant will automatically load the built-in version again.

---

## Support

- **Issues**: [GitHub Issues](https://github.com/GuiPoM/netatmo-plus/issues)
- **pyatmo PR**: [jabesq-org/pyatmo#554](https://github.com/jabesq-org/pyatmo/pull/554)

## License

Based on the Home Assistant Netatmo integration, licensed under Apache License 2.0.
