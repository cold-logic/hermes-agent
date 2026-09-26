# Fork Changelog Ledger

Tracking upstream merges into `cold-logic/hermes-agent`, a fork of
`NousResearch/hermes-agent` with custom Jujutsu workspace integration,
HERMES_HOME_MODE, lifecycle guard, and GHCR publishing.

## Format

Each entry records the date, merge commit, upstream sync point, conflicts
encountered, fork-specific changes verified, and deployment impact.

---

## 2026-09-26 — v2026.9.24 sync

- **Merge commit**: `4c51473059c9` — "Merge upstream/main into fork (v2026.9.26 sync)"
- **Upstream HEAD**: `e29517479e` — "refactor(onboarding): ask for the plain intro once"
- **Upstream release**: `v2026.9.24` (Hermes v0.21.5)
- **Commits merged**: 4,409
- **Previous sync**: 2026-09-22 (v2026.9.21 / v0.21.4)
- **GHCR digest**: `sha256:212e24cd265d6600e1edeb517fd04ef34d5aa30e0a8ecb2d3797d5d90c887131`
- **Running version**: `v0.21.5+2961 (2026.9.24)`

### Conflicts (1)

- `pyproject.toml` — Upstream restructured all dependency pins with
  `python_version >= '3.14'` markers. Resolved by taking upstream side
  (`:theirs`). The fork's `cryptography==50.0.1` and `certifi==2026.5.20`
  were already matched by upstream's new structure.

### Fork-specific changes verified surviving

- jj workspace kind (`VALID_WORKSPACE_KINDS` includes `"jj"`) ✓
- `HERMES_HOME_MODE=0710` in `service_manager.py` ✓
- lifecycle_guard data-file exclusion (`_DATA_EXTENSIONS`, 12 extensions) ✓
- GHCR publishing workflow (`.github/workflows/docker-publish-fork.yml`) ✓
- Auth reset fix — fully absorbed by upstream natively as `status_cleared_ids` ✓

### Build issues encountered and fixed

1. **`pyproject.toml` version is now `0.0.0`** — Upstream moved to deriving
   the runtime version from git tags via `hermes_cli/version_info.py` and
   build-time `install-stamp.json`, instead of a static `pyproject.toml`
   version. Generated a stamp before building:
   ```
   python3 scripts/write_install_stamp.py --output install-stamp.json \
     --commit <sha> --base-version 0.21.5 --distance <N> \
     --source ci --distribution docker --update-mechanism external
   ```

2. **`/usr/local/lib/node_modules` missing** — The new base image no longer
   has this directory (Node.js is now baked in differently). Fixed by adding
   `mkdir -p /usr/local/lib/node_modules` before the npm copy step in the
   deployment Dockerfile.

3. **`uv` not on PATH during build** — uv moved to
   `/opt/hermes/tools/uv-*/uv` in the base image. Fixed both `uv pip install`
   calls in the deployment Dockerfile to use the full glob path.

### Deployment impact

- Deployment Dockerfile required fixes (see above)
- Fork image built locally and pushed directly to GHCR (no Actions credits)
- Deployment image rebuilt, `hermes` container recreated
- All containers healthy

---

## 2026-09-22 — v2026.9.21 sync

- **Merge commit**: `1eeb87493a80` — "Merge upstream/main into fork (v2026.9.22 sync)"
- **Upstream HEAD**: `969872ebaa` — "fmt(js): `npm run fix` on merge (#118897)"
- **Upstream release**: `v2026.9.21` (Hermes v0.21.4)
- **Commits merged**: 3,089
- **Previous sync**: 2026-09-18 (v2026.9.18 / v0.21.3)
- **GHCR digest**: `sha256:7ad94123c1c817224b12f1d8fb8351e57778c20bd37c495733c92762a9bd7a34`
- **Running version**: `v0.21.4 (2026.9.21)`

### Conflicts (0)

Clean merge — no conflicts.

### Fork-specific changes verified surviving

- jj workspace kind ✓
- HERMES_HOME_MODE=0710 ✓
- lifecycle_guard (12 extensions) ✓
- GHCR workflow ✓

---

## 2026-09-18 — v2026.9.18 sync

- **Merge commit**: `05424d07d8` — "Merge upstream/main into fork (v2026.9.18 sync, 2194 commits)"
- **Upstream HEAD**: `27a30d8515`
- **Upstream release**: `v2026.9.14` (Hermes v0.21.3) — no new tag since
- **Commits merged**: 2,194
- **Previous sync**: 2026-09-14 (v2026.9.10 / v0.21.1)
- **GHCR digest**: `sha256:4ed2c302d4147d29f9df0f7ff87c71719a42e535c8037fadc4af1c6b8fb620c5`
- **Running version**: `v0.21.3 (2026.9.14)`

### Conflicts (2)

- `tests/hermes_cli/test_auth_profile_fallback.py` — Upstream deleted this
  test (replaced by `test_auth_profile_isolation.py` with opposite behavior:
  profiles now fail closed instead of falling back to root auth, per #111724).
  Accepted upstream deletion; fork's test was testing removed behavior.
- `uv.lock` — Took upstream version.

### Fork-specific changes verified surviving

- jj workspace kind ✓
- HERMES_HOME_MODE=0710 ✓
- lifecycle_guard (12 extensions) ✓
- GHCR workflow ✓
- Auth reset fix — absorbed by upstream natively as `status_cleared_ids` ✓

### Notes

- Auth reset fix (`clear_status_ids`) was fully absorbed by upstream,
  restructured into `agent/credential_pool_admin.py` with `status_cleared_ids`.
  The fork's version is now redundant.
- All 36 upstream-inherited GitHub Actions workflows disabled to avoid
  burning credits with no available runner. Fork's own publish workflow
  also disabled (local build + push used instead).

---

## 2026-09-14 — v2026.9.10 sync

- **Merge commit**: `c0f9f4b0` — "Merge upstream into fork (v2026.9.10)"
- **Upstream release**: `v2026.9.10` (Hermes v0.21.1)
- **Commits merged**: ~1,313
- **Running version**: `v0.21.1 (2026.9.7)`

### Notes

- Initial fork merge tracked in this deployment's history.
- Fork-specific commits: jj workspace integration, auth reset fix,
  HERMES_HOME_MODE=0710, GHCR workflow, lifecycle guard data-file exclusion.
