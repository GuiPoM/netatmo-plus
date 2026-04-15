# Development & Maintenance Guide

Complete guide for developing and maintaining Netatmo Plus.

## Table of Contents

- [Repository Structure](#repository-structure)
- [Development Workflow](#development-workflow)
- [Key Changes vs Official Integration](#key-changes-vs-official-integration)
- [Regular Maintenance](#regular-maintenance)
- [Creating Releases](#creating-releases)
- [Upstream Contribution Plan](#upstream-contribution-plan)
- [Quick Reference](#quick-reference)

---

## Repository Structure

Two repositories are involved:

1. **`GuiPoM/homeassistant_core`** (fork) — Development and testing
   - Feature branches with full HA test framework
   - All tests run here

2. **`GuiPoM/netatmo-plus`** (this repo) — Distribution via HACS
   - Clean, single-commit history per release
   - Only the integration code, no test infrastructure

3. **`GuiPoM/pyatmo`** (fork) — pyatmo library fork
   - Branch `feature/siren-control` — clean PR for upstream (no workarounds)
   - Branch `feature/siren-app-endpoint` — workaround used by this custom component

**Workflow:** Develop in `homeassistant_core` fork → Sync to `netatmo-plus` → Release

---

## Development Workflow

### Making Changes

All development happens in `GuiPoM/homeassistant_core`:

```bash
cd S:\git\perso\homeassistant_core

# Create/checkout feature branch
git checkout -b feature/netatmo-something

# Make changes in homeassistant/components/netatmo/

# Commit changes
git commit -m "feat: Add ..."
```

### Syncing to netatmo-plus

After development and testing in the fork:

```bash
# Copy changed files from fork to netatmo-plus
cp homeassistant/components/netatmo/<changed_file>.py S:\git\perso\netatmo-plus\custom_components\netatmo\

# In netatmo-plus, amend the release commit or create a new one
cd S:\git\perso\netatmo-plus
git add -A
git commit -m "chore: Sync from homeassistant_core"
```

### Squashing for a clean release

Before releasing a new version, squash all commits into one:

```bash
cd S:\git\perso\netatmo-plus

# Create a clean orphan branch
git checkout --orphan release-v1.x.x
git add -A
git commit -m "Release: Netatmo Plus v1.x.x

- Change 1
- Change 2"

# Replace main with the clean branch
git branch -D main
git branch -m main
git push origin main --force
```

---

## Key Changes vs Official Integration

### Files modified

| File | Change |
|---|---|
| `manifest.json` | Name, codeowners, documentation URL, pyatmo fork requirement, version |
| `const.py` | Added `Platform.SIREN`, siren constants (`CONF_SIREN_*`, `WEB_SESSION_AUTH`, etc.) |
| `config_flow.py` | Added `siren_auth` step in `NetatmoOptionsFlowHandler` |
| `__init__.py` | Initialize `NetatmoWebSessionAuth` from stored token in options |
| `data_handler.py` | Added `NETATMO_CREATE_CAMERA_SIREN` signal for camera category |
| `camera.py` | Fixed `monitoring` attribute: use `device.monitoring` instead of `_monitoring` |
| `strings.json` | Added `siren_auth` step strings and `siren_login_failed` error |
| `translations/en.json` | Same as strings.json for English |

### Files added

| File | Purpose |
|---|---|
| `siren.py` | Siren platform — creates `siren.*` entities for NOC cameras |
| `web_auth.py` | Netatmo web session authentication for siren control |

### pyatmo dependency

`manifest.json` points to `GuiPoM/pyatmo@feature/siren-app-endpoint` which:
- Contains the `SirenMixin` for NOC
- Uses `app.netatmo.net` as the endpoint for siren control (workaround for OAuth2 limitation)

When upstream pyatmo PR #554 is merged and released, update `manifest.json`:
```json
"requirements": ["pyatmo==X.Y.Z"]
```

---

## Regular Maintenance

### Keeping up with official integration updates

Periodically check for updates to the official Netatmo integration:

```bash
# In homeassistant_core fork, fetch upstream
git fetch upstream
git log upstream/dev -- homeassistant/components/netatmo/ --oneline | head -20
```

If there are new commits:

1. Cherry-pick or merge relevant changes
2. Resolve conflicts with our custom changes
3. Sync to `netatmo-plus`
4. Create a new release

### Checking for pyatmo updates

```bash
$env:GH_HOST="github.com"
gh api repos/jabesq-org/pyatmo/releases --jq '.[0] | {tag: .tag_name, date: .published_at}'
```

If a new pyatmo version is released that includes `SirenMixin` (PR #554 merged), update `manifest.json` to use the official version.

---

## Creating Releases

1. **Squash all changes** into a single clean commit (see workflow above)

2. **Update version** in `manifest.json` and `hacs.json`

3. **Push to GitHub**:
   ```bash
   git push origin main --force
   ```

4. **Create a GitHub release**:
   ```bash
   $env:GH_HOST="github.com"
   gh release create v1.0.0 --title "v1.0.0" --notes "Initial release" --repo GuiPoM/netatmo-plus
   ```

5. **Update HACS** — users will see the update automatically

---

## Upstream Contribution Plan

The goal is to eventually upstream all changes:

| Feature | Status | Blocker |
|---|---|---|
| `SirenMixin` in pyatmo | PR #554 submitted | Awaiting review by maintainers |
| Siren platform in HA | Not submitted yet | Waiting for pyatmo PR merge + release |
| `monitoring` fix in HA | Not submitted yet | Simple fix, can be submitted anytime |

### Contact

- **pyatmo maintainer**: @jabesq on GitHub
- **HA Netatmo codeowner**: @cgtobi on GitHub (also on HA Community forum)

---

## Quick Reference

```bash
# Check current netatmo-plus status
git -C S:\git\perso\netatmo-plus log --oneline

# Check pyatmo PR status
$env:GH_HOST="github.com"; gh pr view 554 --repo jabesq-org/pyatmo

# Push netatmo-plus update
git -C S:\git\perso\netatmo-plus push origin main --force

# Create release
$env:GH_HOST="github.com"; gh release create v1.0.0 --repo GuiPoM/netatmo-plus --generate-notes
```
