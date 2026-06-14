# Development & Maintenance Guide

Complete guide for developing and maintaining Netatmo Plus.

## Table of Contents

- [Repository Structure](#repository-structure)
- [Versioning](#versioning)
- [Key Files — Ownership](#key-files--ownership)
- [HA Version Upgrade Procedure](#ha-version-upgrade-procedure)
- [Development Workflow](#development-workflow)
- [Creating Releases](#creating-releases)
- [Upstream Contribution Plan](#upstream-contribution-plan)
- [Quick Reference](#quick-reference)

---

## Repository Structure

Three repositories are involved:

1. **`GuiPoM/homeassistant_core`** (fork) — Development and testing
   - Branch `feature/netatmo-plus-validation` — all our changes + tests
   - Rebased on HA release tags (e.g. `2026.6.0`)
   - All tests run here via Docker devcontainer (`magical_keller`)

2. **`GuiPoM/netatmo-plus`** (this repo) — Distribution via HACS
   - `main` branch — current release
   - Only the integration code, no test infrastructure

3. **`GuiPoM/pyatmo`** (fork) — pyatmo library fork
   - Branch `feature/siren-optional-base-url` — PR #566 merged into upstream
   - `manifest.json` still points to `feature/siren-app-endpoint` until pyatmo releases a version including the merged PR

**Workflow:** Rebase `feature/netatmo-plus-validation` on new HA tag → Run tests → Apply diffs to `netatmo-plus` → Release

---

## Versioning

Version numbers follow the Home Assistant release the integration is based on:

| Release type | Format | Example |
|---|---|---|
| Beta | `YYYY.M.Pb1` | `2026.6.0b1` |
| Stable | `YYYY.M.P` | `2026.6.0` |
| Patch on same HA base | `YYYY.M.P.N` | `2026.6.0.1` |

`hacs.json` `homeassistant` field must match the base HA version.

---

## Key Files — Ownership

**CRITICAL: never copy these files from upstream HA — they are ours:**

| File | Why |
|---|---|
| `siren.py` | Our addition — siren platform for NOC |
| `web_auth.py` | Our addition — Netatmo web session auth |
| `light.py` | Contains our `alim_status` availability fix (see below) |
| `manifest.json` | Our metadata (name, version, pyatmo fork requirement) |
| `hacs.json` | Our HACS config |

**Files we modify (apply upstream diffs on top of our changes, never overwrite):**

| File | Our changes |
|---|---|
| `__init__.py` | `NetatmoWebSessionAuth` init + `async_config_entry_updated` |
| `camera.py` | `monitoring` fix, `light_state` fix, extra attributes |
| `climate.py` | Extra attributes (scheduled_temperature, open_window, etc.) |
| `config_flow.py` | `siren_auth` options flow step |
| `const.py` | `Platform.SIREN`, siren constants, `NETATMO_CREATE_CAMERA_SIREN` |
| `data_handler.py` | `NETATMO_CREATE_CAMERA_SIREN` in camera category |
| `strings.json` | `siren_auth` step strings |
| `translations/en.json` | Same |

**Files we copy verbatim from upstream (no our changes):**

`api.py`, `application_credentials.py`, `binary_sensor.py`, `button.py`, `cover.py`,
`device_trigger.py`, `diagnostics.py`, `entity.py`, `fan.py`, `helper.py`,
`media_source.py`, `select.py`, `sensor.py`, `switch.py`, `webhook.py`

### Known bug: `alim_status` availability fix

`light.py` and `siren.py` override `available` to use `device.alim_status is not None`
instead of `bool(data_handler.webhook)`. This fix was introduced because:
- The upstream code gates availability on webhook registration, which is `False` at
  startup and can take minutes to become `True`
- `async_update_callback` reads `device.floodlight` / `device.siren_status` from the
  **polled** `homestatus` API (since pyatmo 7.0.1 refactor), not from webhook events
- Therefore webhook status is incorrect as an availability proxy

This bug also exists in upstream HA core (tracked for future upstream PR).
**Do not copy `light.py` or `siren.py` from upstream — the fix will be lost.**

---

## HA Version Upgrade Procedure

When a new HA release is out (e.g. `2026.7.0`):

### 1. Fetch new tag and check netatmo changes

```powershell
git -C "S:\git\perso\homeassistant_core" fetch upstream --tags

# Check what changed in netatmo between old and new tag
git -C "S:\git\perso\homeassistant_core" diff --name-only 2026.6.0..2026.7.0 -- homeassistant/components/netatmo/ tests/components/netatmo/
```

### 2. Rebase feature branch on new tag

```powershell
git -C "S:\git\perso\homeassistant_core" checkout feature/netatmo-plus-validation
git -C "S:\git\perso\homeassistant_core" rebase 2026.7.0
```

Resolve conflicts if any (expected on our modified files).

### 3. Run tests in devcontainer

```powershell
docker start magical_keller
docker exec magical_keller bash -c "cd /workspaces/homeassistant_core && /home/vscode/.local/ha-venv/bin/python -m pytest tests/components/netatmo/test_climate.py tests/components/netatmo/test_camera.py tests/components/netatmo/test_config_flow.py -q 2>&1 | tail -20"
```

Update snapshots if needed:
```powershell
docker exec magical_keller bash -c "cd /workspaces/homeassistant_core && /home/vscode/.local/ha-venv/bin/python -m pytest tests/components/netatmo/test_climate.py --snapshot-update -q 2>&1 | tail -10"
```

### 4. Sync to netatmo-plus

**Generate the upstream diff and apply it** — never copy files manually.

```powershell
# Generate patch for all netatmo files changed between the two HA tags
git -C "S:\git\perso\homeassistant_core" diff 2026.6.0..2026.7.0 `
    -- homeassistant/components/netatmo/ `
    > S:\temp\netatmo-upstream.patch

# Review the patch — check which of our files are touched
git -C "S:\git\perso\homeassistant_core" diff --name-only 2026.6.0..2026.7.0 `
    -- homeassistant/components/netatmo/

# Apply the patch to netatmo-plus
# The patch paths are homeassistant/components/netatmo/* — strip 3 levels
# to land in custom_components/netatmo/*
git -C "S:\git\perso\netatmo-plus" apply `
    --directory=custom_components/netatmo `
    -p3 S:\temp\netatmo-upstream.patch

Remove-Item "S:\temp\netatmo-upstream.patch"
```

If `git apply` reports conflicts on our modified files (`__init__.py`, `camera.py`,
`climate.py`, `config_flow.py`, `const.py`, `data_handler.py`), resolve them manually
using the patch as reference — **never overwrite, always merge**.

**NEVER let git apply touch these files** (our additions/fixes) — reject those hunks:
- `siren.py`, `web_auth.py`, `light.py`, `manifest.json`, `hacs.json`

### 5. Update version and push

```powershell
# manifest.json: bump version to 2026.7.0b1
# hacs.json: bump homeassistant to 2026.7.0
git -C "S:\git\perso\netatmo-plus" add -A
git -C "S:\git\perso\netatmo-plus" commit -m "chore: rebase on HA 2026.7.0 — bump to v2026.7.0b1"
git -C "S:\git\perso\netatmo-plus" push origin main
```

### 6. Create beta release, validate, then stable

```powershell
$env:GH_HOST="github.com"
gh release create v2026.7.0b1 --repo GuiPoM/netatmo-plus --title "v2026.7.0b1 (beta)" --prerelease --target main --notes "Based on HA 2026.7.0 — beta"
# ... validate on real device ...
gh release delete v2026.7.0b1 --repo GuiPoM/netatmo-plus --yes
# bump manifest to 2026.7.0, commit, push
gh release create v2026.7.0 --repo GuiPoM/netatmo-plus --title "v2026.7.0" --target main --notes-file "S:\temp\gh-body.md"
```

---

## Development Workflow

### Making changes

All development in `GuiPoM/homeassistant_core` on `feature/netatmo-plus-validation`:

```powershell
git -C "S:\git\perso\homeassistant_core" checkout feature/netatmo-plus-validation
# edit files in homeassistant/components/netatmo/
git -C "S:\git\perso\homeassistant_core" commit -m "feat: ..."
```

### Docker devcontainer

- Container: `magical_keller`
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

---

## Creating Releases

1. Ensure all changes are committed and pushed to `main`
2. Update `manifest.json` version and `hacs.json` homeassistant field
3. Update `CHANGELOG.md`
4. Create GitHub release via `gh release create`

---

## Upstream Contribution Plan

| Feature | Status |
|---|---|
| `SirenMixin` in pyatmo | Merged in pyatmo v9.3.0 |
| `base_url` override for siren | PR #566 merged into pyatmo `development` — awaiting release |
| Siren platform in HA core | Pending pyatmo release with #566 |
| `monitoring` fix in HA core | Not submitted yet |
| `light.py` availability fix in HA core | Not submitted yet (bug exists upstream) |
| Climate extra attributes | Not submitted yet |
| Camera extra attributes | Not submitted yet |

### Contacts

- **pyatmo maintainer**: @cgtobi on GitHub
- **HA Netatmo codeowner**: @cgtobi on GitHub

---

## Quick Reference

```powershell
# Check netatmo changes between HA releases
git -C "S:\git\perso\homeassistant_core" diff --name-only 2026.6.0..2026.7.0 -- homeassistant/components/netatmo/

# Run netatmo tests
docker exec magical_keller bash -c "cd /workspaces/homeassistant_core && /home/vscode/.local/ha-venv/bin/python -m pytest tests/components/netatmo/test_climate.py tests/components/netatmo/test_camera.py tests/components/netatmo/test_config_flow.py -q 2>&1 | tail -20"

# Check latest pyatmo release
$env:GH_HOST="github.com"; gh api repos/jabesq-org/pyatmo/releases --jq '.[0] | {tag: .tag_name, date: .published_at}'

# Create beta release
$env:GH_HOST="github.com"; gh release create v2026.X.Yb1 --repo GuiPoM/netatmo-plus --prerelease --target main --title "v2026.X.Yb1 (beta)" --notes "..."

# Create stable release
$env:GH_HOST="github.com"; gh release create v2026.X.Y --repo GuiPoM/netatmo-plus --target main --title "v2026.X.Y" --notes-file "S:\temp\gh-body.md"
```
