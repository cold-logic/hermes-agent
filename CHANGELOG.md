# Fork Changelog Ledger

Tracking upstream merges into `cold-logic/hermes-agent`, a fork of
`NousResearch/hermes-agent` with custom Jujutsu workspace integration,
HERMES_HOME_MODE, lifecycle guard, and GHCR publishing.

## Format

Each entry records the date, merge commit, upstream sync point, conflicts
encountered, fork-specific changes verified, and deployment impact.

---

## 2026-10-10 — sync to upstream `1f711b27`; Browser Use CLI becomes default browser surface

- **Merge commit**: `c39f38ab95b29cc8510992381536129adfe84340` — "Merge
  upstream/main into fork (2026-10-10 sync)"
- **Upstream HEAD**: `1f711b275f` — "test(desktop): the source hand-off fixture
  installs its loopback transport once"
- **Upstream release**: `v0.21.6` tag exists but its `pyproject.toml` still reads
  `0.0.0` → `v2026.9.24` (`0.21.5`) remains the install-stamp base.
- **Commits merged**: 404 (behind 404 → 0; ahead 28 → 29 with the merge)
- **GHCR digest**: `sha256:ac9b923762af0d47a4a1217af088dafc9aefc6b6c7c4dba71fd403696d11b720`
  (locally built `hermes-agent-fork-test` `fc022181`, tagged + pushed)
- **Running version**: `v0.21.5+9744 (2026.9.24) · upstream c39f38ab`
- **Pushed**: `git push origin main` → `b6ec2221fe..c39f38ab95`

### Conflicts

**One**: `hermes_cli/kanban_db_workspace.py` — the late-bound import block at
EOF. Upstream kept `from hermes_cli import kanban_db as _kb`; the fork side adds
`from hermes_cli.kanban_db_notify import add_notify_sub`, which `_jj_notify`
calls (module present in merged tree). Resolved keeping the fork side.

Merge-vs-upstream diff is exactly the 9-file fork payload.

### Fork-specific changes verified surviving

- jj workspace kind (`VALID_WORKSPACE_KINDS` includes `"jj"`) ✓
- `HERMES_HOME_MODE=0710` in `service_manager.py` ✓
- lifecycle_guard `_DATA_EXTENSIONS` (12 entries) ✓
- GHCR publishing workflow (`.github/workflows/docker-publish-fork.yml`) ✓
- Upstream auth reset (`status_cleared_ids`) present ✓
- jj symbol map: all three `def`s still in `kanban_db_workspace.py`; every
  `_JJ_CANDIDATES`/`_JJ2` list keeps `/usr/local/bin/jj` first ✓
- install-stamp regenerated on disk to `0.21.5+9744` (gitignored, not committed —
  see 2026-10-08 mechanics note) ✓

### Deployment impact (this pass)

- Pin bumps: node `26-bookworm-slim` digest, ollama digest, nmem 0.10.99,
  mise 2026.10.7, agy 1.3.3, devin 3000.11.51, jj 0.46.0 (10-08 deferral
  cleared — now in mise registry). tmux 3.8 + direnv 2.38.2 still deferred.
- **New upstream behavior — Browser Use CLI is the default browser surface**:
  with `browser.backend` unset and `browser-harness` importable (it ships in the
  PM-sealed env), `is_browser_use_cli_mode()` is true and `browser_exec`
  (`tools/browser_use_cli.py`, `check_fn=is_browser_use_cli_mode`) replaces the
  whole `browser_*` toolset — the `_browser_cdp_check → False` log lines are
  correct gating, not a broken sidecar. The deployment's `browser.cdp_url`
  sidecar still backs `browser_exec` sessions. `browser.backend: off` in
  `config.yaml` restores the legacy tools if ever wanted.
- Zulip adapter (volume-side plugin) needed `python_dependencies:
  ["zulip>=0.9.0,<0.11"]` declared in `/opt/data/plugins/zulip/plugin.yaml` —
  the gateway's sealed PM env never had the SDK (Dockerfile installs it only
  into the unused base venv). Env rebuilt at boot → `✓ zulip connected`.

## 2026-10-08 — sync to upstream `38880bd2`; carried test patch absorbed

- **Merge commit**: `64a2471592` — "Merge upstream/main into fork (2026-10-08 sync)"
- **Upstream HEAD**: `38880bd2` — "fix(s6): read the supervised pid from
  s6-svstat -o, not the status line"
- **Upstream release**: `v0.21.6` tag exists now, but its `pyproject.toml` still
  reads `0.0.0` → `v2026.9.24` (`0.21.5`) remains the install-stamp base.
- **Commits merged**: 956 (behind 956 → 0; ahead 25 → 26 with the merge)
- **GHCR digest**: `sha256:8cea979a5e402185cec415c1c126df0c4d2f635157f038623dc5da21721ffd40`
  (locally built `hermes-agent-fork-test` `613394a9`, tagged + pushed)
- **Running version**: `v0.21.5+9337 (2026.9.24) · upstream 64a24715`
- **Pushed**: `git push origin main` → `7fda32aeba..64a2471592` (merge), then
  `64a2471592..d73ff9be93` (empty stamp-marker commit — see note)

### Conflicts

**One**: `web/src/components/ChatSessionList.test.tsx` (2-sided). Upstream
`eb6a3886f1` applied the same Button-mock typecheck fix our carried patch
`fdbc23daf0` did (identical props; theirs single-line, ours multi-line).
Resolved `:theirs` — **the carried patch is now fully absorbed; zero fork delta
on that file**. The 10-06 "drop when upstream fixes" condition is met.

Merge-vs-upstream diff is exactly the 9-file fork payload (+1,371/−5).

### install-stamp mechanics (learned this sync)

`install-stamp.json` is **gitignored upstream** (`.gitignore:278`, present before
this merge) and was never committed in fork history — the Oct-6 session's
`git ls-files` hit was stale index. It reaches the image anyway: the local
`docker build` context (`COPY --link . .`, not dockerignored) picks it up off
disk. The `d73ff9be` "chore: refresh install-stamp" commit pushed **empty**
because jj never tracks the ignored file — regenerate the stamp on disk before
building; don't commit it. Fork GHCR builds are local `docker push` (Actions
queued indefinitely — all disabled), so nothing else consumes the stamp.

### Fork-specific changes verified surviving

- jj workspace kind (`VALID_WORKSPACE_KINDS` includes `"jj"`, kanban_db.py:115,
  validated :1234) ✓
- `HERMES_HOME_MODE=0710` in `service_manager.py:448` ✓
- lifecycle_guard `_DATA_EXTENSIONS` (:308, 12 entries; applied :1011) ✓
- GHCR publishing workflow (`.github/workflows/docker-publish-fork.yml`) ✓
- Upstream auth reset (`status_cleared_ids`,
  `agent/credential_pool_admin.py`) present ✓
- jj symbol map: all three `def`s still in `kanban_db_workspace.py`
  (`_jj_notify` :361, `_cleanup_workspace` :436, `resolve_workspace` :1041);
  every `_JJ_CANDIDATES`/`_JJ2` list (2 in `kanban_db_workspace.py`, 4 in
  `kanban.py`) keeps `/usr/local/bin/jj` first ✓
- Deployment-side pins also bumped this pass: mise 2026.10.4, agy 1.3.1,
  devin 3000.11.39, direnv 2.38.1, pingap 0.15.0, nmem 0.10.97, ollama +
  chromedp digest re-pins (deployment repo commit separately)

## 2026-10-06 — post-v2026.9.24 sync (+ carried web typecheck patch)

- **Merge commit**: `7232a0bfc367` — "Merge upstream/main into fork (2026-10-06 sync)"
- **Carried patch**: `fdbc23daf077` — "fix(dashboard): typecheck-clean
  ChatSessionList test mock (fork)". Upstream merged
  `web/src/components/ChatSessionList.test.tsx` today (`39ac8f2b`, touched again
  by `da1bf11`) with two errors the Docker web build's solution-builder
  typecheck rejects: bare `globalThis.IS_REACT_ACT_ENVIRONMENT` (TS7017) and
  `ghost`/`outlined`/`size` destructured off `ButtonHTMLAttributes` (TS2339).
  Fixed matching sibling-test conventions (globalThis cast + extended mock prop
  type). **Drop when upstream fixes the file.**
- **Upstream HEAD**: `04dc1c6e` — "fix(lint): stdin guard reads only real stdin
  settings; unbroker key generation leaves no partial file (review)"
- **Upstream release**: `v2026.9.24` remains the last tag with a real
  `pyproject.toml` version (`0.21.5`). Upstream's tag scheme changed: daily
  `v0.21.4+canary.<date>` + `rc.NN-v0.21.5` RCs — the `v20*` glob is dead.
- **Commits merged**: 1,018 (behind went 2,432 → 0 across yesterday+today)
- **GHCR digest**: `sha256:e1637299630c3b843b195b5ae25a4dfb6f369703e3ac71e37043724510300c25`
  (pushed `ghcr.io/cold-logic/hermes-agent:latest`)
- **Running version**: `v0.21.5+8379 (2026.9.24) · upstream fdbc23da`
- **Pushed**: `git push origin main` → `d93853918c..7232a0bfc3` (merge), then
  `7232a0bfc3..fdbc23daf0` (patch)

### Conflicts

None — clean merge (upstream touched ~1,200 files; zero overlap with the fork
payload). Diff of merge commit vs upstream tip is exactly the 9-file fork delta
(+1,302, incl. yesterday's CHANGELOG entry).

Bookkeeping note: `main` again arrived as a conflicted bookmark after fetch
(local tip `d9385391` vs upstream tip `04dc1c6e` vs stale `b4bf19d8`); resolved
by `jj bookmark set main -r d9385391` before creating the merge.

### Fork-specific changes verified surviving

- jj workspace kind (`VALID_WORKSPACE_KINDS` includes `"jj"`, kanban_db.py:115) ✓
- `HERMES_HOME_MODE=0710` in `service_manager.py:448` ✓
- lifecycle_guard `_DATA_EXTENSIONS` (:308, 12 entries; applied :1011) ✓
- GHCR publishing workflow (`.github/workflows/docker-publish-fork.yml`) ✓
- Upstream auth reset (`status_cleared_ids`,
  `agent/credential_pool_admin.py:30,53`) present ✓
- jj symbol map: all three `def`s still in `kanban_db_workspace.py`; every
  `_JJ_CANDIDATES`/`_JJ2` list (6 sites — 2 in `kanban_db_workspace.py`, 4 in
  `kanban.py`) keeps `/usr/local/bin/jj` first. Line numbers drifted;
  `stages/02_fork_sync/references/fork-internals.md` in the deployment repo
  refreshed ✓

### Build notes

- Install stamp regenerated for `fdbc23da` — distance 8,379 from `v2026.9.24`
  (`0.21.5+8379`; counts upstream + fork commits since the tag — expected).
- First `docker build` FAILED at `frontend_build` (the upstream test-file
  typecheck above); rebuild after `fdbc23da` was clean.
- Same pass bumped deployment pins: node `26-bookworm-slim` digest
  (`3ffc19ea…`), ollama digest (`1bef6397…`), nmem 0.10.96, agy 1.3.0, devin
  3000.11.35. direnv 2.38.1 deferred (released same-day; mise aqua registry
  lag — `mise ls-remote direnv` topped at 2.37.1).

### Deployment impact

- `hermes`, `nowledge-mem`, `ollama` recreated; `tailscale`/`browser`
  untouched. All s6 longruns up, 6/6 overmind procs running, all gateway
  platforms (webhook/api_server/telegram/discord/mattermost/zulip) connected.
- New upstream behavior surfaced: `left_core_migration` auto-installs left-core
  plugins — `homeassistant` install denied by the root-owned
  `/opt/data/plugins` tree (run `hermes plugins install homeassistant` if HA is
  wanted). New `fallback_providers` validation warns the deployed config.yaml
  stores it as a quoted string (pre-existing; every reader ignores it).

## 2026-10-05 — post-v2026.9.24 sync

- **Merge commit**: `144e9f9b197d` — "Merge upstream/main into fork (2026-10-05 sync)"
- **Upstream HEAD**: `b4bf19d8148a` — "fix(desktop): Review scope tabs get their own row and a narrow dropdown"
- **Upstream release**: still `v2026.9.24` (Hermes v0.21.5) — no new tag; all
  commits are pre-release work since the tag
- **Commits merged**: 2,432
- **Previous sync**: 2026-09-30 (post-v2026.9.24 / v0.21.5+4924)
- **GHCR digest**: none — image built locally and tagged
  `ghcr.io/cold-logic/hermes-agent:latest` **but not pushed** this pass
  (build-local-only choice); registry `latest` still points at the 2026-09-30 image
- **Running version**: `v0.21.5+7358 (2026.9.24) · upstream 144e9f9b`
- **Pushed**: `git push origin main` → `a59b70ac12..144e9f9b19`

### Conflicts

None — clean merge despite the size (1,706 upstream files touched; zero overlap
with the fork payload). Diff of merge commit vs upstream tip is exactly the
9-file fork delta (+1,249).

Bookkeeping note: `main` arrived as a three-way conflicted bookmark after fetch
(local tip vs upstream tip vs the stale `f42f579c` position); resolved by
`jj bookmark set main -r <merge>` per the standard recipe.

### Fork-specific changes verified surviving

- jj workspace kind (`VALID_WORKSPACE_KINDS` includes `"jj"`, kanban_db.py:115) ✓
- `HERMES_HOME_MODE=0710` in `service_manager.py` ✓
- lifecycle_guard data-file exclusion (`_DATA_EXTENSIONS`) ✓
- GHCR publishing workflow (`.github/workflows/docker-publish-fork.yml`) ✓
- Upstream auth reset (`status_cleared_ids` in `agent/credential_pool_admin.py`)
  present — fork's redundant version stays retired ✓
- jj symbol map unchanged (still `kanban_db_workspace.py` + `kanban.py`); both
  `_JJ_CANDIDATES`/`_JJ2` lists keep `/usr/local/bin/jj` first ✓

### Build notes

- Install stamp regenerated for merge commit `144e9f9b` (distance 7,358 from
  `v2026.9.24`). `+7358` counts upstream + fork commits since the tag — expected.
- Same pass bumped deployment pins: mise 2026.10.3, agy 1.2.17, devin
  3000.11.31 (noncontiguous publish — 3000.11.30/.31 exist, neighbors 403),
  nub 0.9.6 (kept the native-ELF tree layout), nmem 0.10.95, ollama digest.

### Deployment impact

- Fork image `hermes-agent-fork-test:latest` (`9d7a249848c7…`) built locally,
  tagged `ghcr.io/cold-logic/hermes-agent:latest` locally — **not pushed to
  GHCR**; do `docker push ghcr.io/cold-logic/hermes-agent:latest` when ready
- Deployment image rebuilt (`e66228a25883…`), `hermes`/`nowledge-mem`/`ollama`
  recreated; all services healthy, gatekeeper active, gateway running under s6

---

## 2026-09-30 — post-v2026.9.24 sync

- **Merge commit**: `e4e9c07953a0` — "Merge upstream/main into fork (v2026.9.30 sync)"
- **Upstream HEAD**: `f42f579cf8` — "Merge pull request #128011 from rroverin/fix/escape-drift-newline-doubling"
- **Upstream release**: still `v2026.9.24` (Hermes v0.21.5) — no new tag; all
  commits are pre-release work since the tag
- **Commits merged**: 1,960
- **Previous sync**: 2026-09-26 (v2026.9.24 / v0.21.5)
- **GHCR digest**: `sha256:2c8fb6738e7950b06d1f915a1bf3b52b0f47cbeb0412fcdfdd74a231c628835a`
- **Running version**: `v0.21.5+4924 (2026.9.24)`

### Conflicts

None — clean merge.

### Fork-specific changes verified surviving

- jj workspace kind (`VALID_WORKSPACE_KINDS` includes `"jj"`) ✓
- `HERMES_HOME_MODE=0710` in `service_manager.py` ✓
- lifecycle_guard data-file exclusion (`_DATA_EXTENSIONS`) ✓
- GHCR publishing workflow (`.github/workflows/docker-publish-fork.yml`) ✓

### Build notes

- Install stamp regenerated for merge commit `e4e9c079` (distance 4,924 from
  `v2026.9.24` tag). `+4924` counts all upstream commits plus the fork-specific
  commits and merge commits — expected, see 2026-09-26 entry.
- Deployment-side issue: **nub 0.9.5's package layout changed** (mise now ships
  it as a native ELF binary tree `bin/{nub,nubx,nubr,nub-launcher-*}` +
  `runtime/` instead of the npm-backed `@nubjs/nub` JS launcher). The
  deployment Dockerfile's `NODE_PATH` wrapper scripts broke
  (`MODULE_NOT_FOUND`). Fixed by copying the tree to `/usr/local/lib/nub` and
  symlinking `bin/{nub,nubx}` into `/usr/local/bin`. Fixed in deployment repo,
  not this fork.

### Deployment impact

- Fork image built locally, tagged `ghcr.io/cold-logic/hermes-agent:latest`
  locally, then pushed to GHCR (no Actions credits)
- Deployment image rebuilt, `hermes` container recreated
- All containers healthy

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
