# PHASE 3 – Update System Verification (Moltbot HA Add-on v1.0.0)

This document is a **verification checklist** for the Phase 3 update system:
- isolated builds
- smoke tests
- caching + activation
- rollback safety (old version stays active on failure)
- update modes + version pinning

---

## Update flow (high-level)

```text
Boot
  ↓
1) Migrate legacy layout (if needed)
  ↓
2) Detect current version (from /config/moltbot/.meta/current_version)
  ↓
3) Determine desired version
   - pinned_version takes priority
   - otherwise: check_for_updates() based on update_mode
  ↓
4) If version missing in cache: download_and_build_version()
   - isolated build dir
   - install deps + build gateway + build UI
   - smoke test
   - on success: move into cache/<version>
  ↓
5) Activate version (activate_version)
   - update symlink: /config/moltbot/source/active -> /config/moltbot/cache/<version>
  ↓
6) Start gateway from active version
  ↓
7) Moltbot setup (first install only)
```

### Key changes vs. the old approach

- ❌ **Old:** `git pull` + `git reset --hard` + build “in place”
- ✅ **New:** isolate build → smoke test → cache → activate
- ✅ Failure handling: if anything fails, the **previous version remains active**
- ✅ Version tracking lives in `/config/moltbot/.meta/`

---

## Verification checklist

### Manual tests on a Home Assistant system

#### Test 1: Fresh install

```bash
# 1) Install the add-on (first installation)
# 2) Check logs
ha addons logs local_moltbot

# Expected log hints (examples):
# [addon] update_mode=stable
# [addon] no current version, detected: <version>
# [addon] using version: <version>
# [addon] downloading and building version <version>
# [addon] installing dependencies for <version>
# [addon] building gateway for <version>
# [addon] building UI for <version>
# [addon] smoke testing <version>
# [addon] version <version> built and cached successfully
# [addon] activated version <version>

# 3) Check directories
ssh -p 2222 root@<HA-IP>
ls -la /config/moltbot/
ls /config/moltbot/cache/               # Should contain at least one version
ls /config/moltbot/.meta/
cat /config/moltbot/.meta/current_version
ls -l /config/moltbot/source/active     # Symlink -> cache/<version>
```

#### Test 2: Update modes

```bash
# 2a) disabled mode
# 1) Set add-on config: update_mode = "disabled"
# 2) Restart add-on
# 3) Check logs for update decisions
ha addons logs local_moltbot | grep -i update
# Expected: no update attempts, stays on current version

# 2b) notify mode
# 1) Set update_mode = "notify"
# 2) Make a newer version available upstream
# 3) Restart add-on
# Expected: "update available: <old> → <new>, awaiting approval"

# 2c) stable mode (default)
# 1) Set update_mode = "stable"
# 2) Publish a new stable tag (no alpha/beta/rc)
# 3) Restart add-on
# Expected: automatic update to the new stable version

# 2d) latest mode
# 1) Set update_mode = "latest"
# 2) Publish a pre-release tag (alpha/beta/rc)
# 3) Restart add-on
# Expected: updates can include pre-releases
```

#### Test 3: Version pinning

```bash
# 1) Set add-on config:
#    - update_mode = "stable"
#    - pinned_version = "v0.2.14"
# 2) Restart add-on
# 3) Check logs
ha addons logs local_moltbot | grep -i pinned
# Expected: pinned_version is acknowledged and used (if available)

# 4) Verify current_version
ssh -p 2222 root@<HA-IP>
cat /config/moltbot/.meta/current_version
# Should be "v0.2.14"
```

#### Test 4: Cache management

```bash
# 1) Ensure multiple versions exist (via repeated updates)
# 2) Set max_cached_versions = 2
# 3) Restart after a third version exists
ssh -p 2222 root@<HA-IP>
ls /config/moltbot/cache/
# Expected: only 2 versions kept (newest + one older), plus current stays protected

# 4) Check logs for cleanup messages
ha addons logs local_moltbot | grep -i "removing old cached"
```

#### Test 5: Smoke test failure handling

Goal: ensure a “broken” version is **not activated**.

```bash
# Strategy:
# - make a version that fails smoke test
# - pin to that version
# - verify the add-on stays on the previous working version

# Expected log hints:
# [addon] smoke test failed for <version>
# [addon] staying on <old_version>
```

(Exact steps depend on your upstream workflow and whether you can publish a broken tag.)

---

## Automated syntax checks

```bash
# Bash syntax
bash -n moltbot_gateway/run.sh
# No output = OK

# JSON syntax
jq . moltbot_gateway/config.json
# Pretty-printed JSON = OK
```

---

## Success criteria (Phase 3)

### Implementation
- [x] `config.json` contains all update options (update_mode, pinned_version, cache limits)
- [x] Version tracking functions implemented
- [x] Isolated build + smoke test pipeline implemented
- [x] Cache + activation logic implemented
- [x] Update logic integrated into `run.sh`

### Manual verification (on HA)
- [ ] Fresh install works end-to-end
- [ ] `disabled` mode works
- [ ] `notify` mode detects updates without applying them automatically
- [ ] `stable` mode updates automatically to stable tags only
- [ ] `latest` mode can include pre-releases
- [ ] Version pinning works
- [ ] Cache cleanup removes older versions as configured
- [ ] Smoke test failures prevent activation
- [ ] Rollback safety: on failure, old version remains active

### Performance targets
- [ ] First install build: < 15 minutes
- [ ] Update (with cache): < 10 minutes
- [ ] Smoke test: < 10 seconds
- [ ] No lingering temp directories / no growth over time

---

## Known limitations

- `notify` mode may require HA notifications (planned for a later phase).
- Disk usage grows with cached builds (mitigated via `max_cached_versions` and `auto_cleanup_versions`).
- Tag formats are expected to follow `v1.2.3` (pre-release detection uses `alpha|beta|rc`).

---

## User-facing docs snippet

### Update modes

| Mode | Behavior | Typical use |
|------|----------|-------------|
| `disabled` | No updates | manual control / development |
| `notify` | Detect updates, wait for approval | user chooses when to update |
| `stable` | Auto-update stable releases only | **default** for production |
| `latest` | Auto-update including pre-releases | testing / early adopters |

### Example add-on configuration

```yaml
update_mode: stable
pinned_version: ""              # empty = newest allowed version
auto_cleanup_versions: true
max_cached_versions: 2          # current + one previous (rollback)
```

---

## Suggested commit message

```text
feat(update-system): implement complete update system with rollback (PHASE 3)

- Add update-mode config options (disabled|notify|stable|latest)
- Implement version-tracking functions (get/set current version)
- Add isolated build system with smoke-tests
- Implement cache management with cleanup
- Add automatic update-check and activation
- Support version-pinning for manual control
```
