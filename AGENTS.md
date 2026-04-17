# Netatmo Plus — Agent Instructions

## Release Checklist

**Before every release, always:**

1. **Update version** in `custom_components/netatmo/manifest.json`

2. **Verify imports** — copy modified files to the Docker devcontainer and run import checks:
   ```bash
   docker cp <file> magical_keller:/tmp/<file>
   # Then run syntax/import check via check script
   ```

2. **Run tests** in the devcontainer:
   ```bash
   docker exec magical_keller bash -c "cd /workspaces/homeassistant_core && python -m pytest tests/components/netatmo/test_climate.py tests/components/netatmo/test_camera.py tests/components/netatmo/test_config_flow.py -q 2>&1 | tail -10"
   ```
   Update snapshots if needed (`--snapshot-update`), then re-run without the flag.

3. **Update CHANGELOG.md** — move items from `[Unreleased]` to the new version section with date.

4. **Update README.md** — reflect any new features or attribute changes in the feature tables.

5. **Create the GitHub release** with `gh release create vX.Y.Z`.

## Repository Structure

- `custom_components/netatmo/` — the integration code (fork of HA core Netatmo)
- `CHANGELOG.md` — release history, always kept up to date
- `README.md` — user-facing documentation, always reflects current features
- `DEVELOPMENT.md` — developer workflow guide

## Related Repositories

- **`GuiPoM/pyatmo`** — fork of pyatmo library
  - Branch `feature/siren-control` — clean PR for upstream (no workarounds)
  - Branch `feature/siren-app-endpoint` — workaround used by this custom component
  - PR upstream: `jabesq-org/pyatmo#554`
- **`GuiPoM/homeassistant_core`** — fork of HA core
  - Branch `feature/netatmo-plus-validation` — tests and snapshots for all changes

## Docker Devcontainer

- Container: `magical_keller`
- Image: `vsc-homeassistant_core-*`
- HA core path: `/workspaces/homeassistant_core`
- Start if needed: `docker start magical_keller`

## Key Design Decisions

- **Siren control** uses Netatmo web session API (`app.netatmo.net`) — the public OAuth2 API rejects `siren_status`
- **Web session credentials** (email + password) stored in `entry.options` — same security level as OAuth2 tokens in HA
- **Re-login** is automatic and transparent — triggered only when a siren command fails with code 3 (token expired)
- **Siren availability** is based on `alim_status` (camera powered), not webhook status
