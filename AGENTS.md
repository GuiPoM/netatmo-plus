# Netatmo Plus — Agent Instructions

## Release Checklist

**Before every release, always:**

1. **Update version** in `custom_components/netatmo/manifest.json` — format `YYYY.M.P` (stable) or `YYYY.M.Pb1` (beta) or `YYYY.M.P.N` (patch).
   Update `hacs.json` `homeassistant` field to match the base HA version.

2. **Run tests** in the devcontainer:
   ```powershell
   docker start magical_keller
   docker exec magical_keller bash -c "cd /workspaces/homeassistant_core && /home/vscode/.local/ha-venv/bin/python -m pytest tests/components/netatmo/test_climate.py tests/components/netatmo/test_camera.py tests/components/netatmo/test_config_flow.py -q 2>&1 | tail -10"
   ```
   Update snapshots if needed (`--snapshot-update`), then re-run without the flag.

3. **Update CHANGELOG.md** — add new version section with date.

4. **Create the GitHub release** with `gh release create vX.Y.Z --repo GuiPoM/netatmo-plus --target main --notes-file S:\temp\gh-body.md`

5. **NEVER delete a published release** — users may have installed it. Edit notes instead.

## Repository Structure

- `custom_components/netatmo/` — the integration code (strict fork of HA core Netatmo)
- `CHANGELOG.md` — release history, always kept up to date
- `README.md` — user-facing documentation
- `DEVELOPMENT.md` — complete developer workflow guide (read this first)

## Versioning

Version numbers follow the Home Assistant release:

| Type | Format | Example |
|---|---|---|
| Stable | `YYYY.M.P` | `2026.6.0` |
| Beta | `YYYY.M.Pb1` | `2026.6.0b1` |
| Patch | `YYYY.M.P.N` | `2026.6.0.2` |

## Related Repositories

- **`GuiPoM/pyatmo`** — fork of pyatmo library
  - Branch `feature/siren-optional-base-url` — PR #566 merged into upstream `jabesq-org/pyatmo`
  - Branch `feature/siren-app-endpoint` — still used by this custom component (manifest.json requirement)
  - Waiting for a new pyatmo release to switch `manifest.json` to official `pyatmo==X.Y.Z`
- **`GuiPoM/homeassistant_core`** — fork of HA core
  - Branch `feature/netatmo-plus-validation` — rebased on HA release tags, tests and snapshots

## Docker Devcontainer

- Container: `magical_keller`
- Image: `vsc-homeassistant_core-*`
- HA core path inside container: `/workspaces/homeassistant_core`
- Python venv: `/home/vscode/.local/ha-venv/bin/python`

If container fails to start (stale WSL socket):
```powershell
docker rm magical_keller
docker run -d --name magical_keller `
  -v "S:\git\perso\homeassistant_core:/workspaces/homeassistant_core" `
  -v "vscode:/vscode" `
  <image_name> sleep infinity
```

## Key Design Decisions

- **Strict upstream alignment** — zero code that doesn't exist in HA core, except our functional additions
- **Our functional additions** (never copy from upstream, never overwrite):
  - `siren.py` — siren platform for NOC camera
  - `web_auth.py` — Netatmo web session auth for siren control
  - `light.py` — contains `alim_status` availability fix (see below)
- **Siren control** uses Netatmo web session API (`app.netatmo.net`) — the public OAuth2 API rejects `siren_status`
- **Web session credentials** (email + password) stored in `entry.options`
- **Re-login** is automatic and transparent — triggered only when a siren command fails with code 3 (token expired)
- **Siren/light availability** is based on `alim_status` (camera powered), not webhook status

## Known Bugs vs Upstream HA

### `light.py` and `siren.py` — availability based on `alim_status`

Upstream HA uses `bool(data_handler.webhook)` for `NetatmoCameraLight.available`.
This was correct when `async_update_callback` relied on webhook events, but since
the pyatmo 7.0.1 refactor it reads from the polled API — making the webhook flag
incorrect. Our fix uses `device.alim_status is not None` instead.

This bug exists in upstream HA core. A PR will be submitted upstream when the
full set of netatmo improvements is contributed to HA core.

## HA Version Upgrade Procedure

See `DEVELOPMENT.md` for the full procedure. Key rule:

**Use `git diff OLD_TAG..NEW_TAG | git apply` — never copy files manually.**

## Lessons Learned (2026.6.0 upgrade)

The 2026.6.0 upgrade was done by manually copying files, which caused:
1. Forgotten files (`webhook.py`, `helper.py`) → import errors on startup
2. Our files overwritten (`siren.py`, `light.py`) → availability regression
3. Unnecessary revert of `device_type_to_str` → unique_id divergence from upstream

Result: 3 patch releases (`v2026.6.0`, `v2026.6.0.1`, `v2026.6.0.2`) and manual
entity migration required for users.

**Fix:** the sync procedure now uses `git diff` + `git apply` (documented in `DEVELOPMENT.md`).

## `device_type_to_str` and unique_id history

- **pyatmo ≤ 9.2.3**: `DeviceType(str, Enum)` → `str(DeviceType.NOC)` = `"DeviceType.NOC"`
- **pyatmo ≥ 9.3.0**: `DeviceType(StrEnum)` → `str(DeviceType.NOC)` = `"NOC"`
- **HA 2026.6.0**: introduced `device_type_to_str` to restore `"DeviceType.NOC"` format
  and maintain backwards compatibility for users who had HA before pyatmo 9.3.0
- **netatmo-plus**: all users installed after pyatmo 9.3.0, so had `"NOC"` format —
  upgrading to 2026.6.0.2 requires manual entity migration (one-time)
