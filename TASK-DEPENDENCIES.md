# Task Dependencies – Moltbot HA Add-on v1.0.0

This document outlines the **dependencies between tasks** and the recommended implementation order.

---

## Dependency graph

```text
REPO SETUP (Tasks 2–4)
├─ Task 2: Remove AGENTS.md
├─ Task 3: Update CLAUDE.md
└─ Task 4: Adjust repository.json
        ↓
PHASE 1: DOCKERFILE (Tasks 5–8)
├─ Task 5: Install Bun via npm (TypeScript fix)  ← CRITICAL
├─ Task 6: Remove duplicates
├─ Task 7: Version pinning
└─ Task 8: Multi-arch build test
        ↓  (blocks everything below)
PHASE 2: STORAGE (Tasks 9–12)
├─ Task 9: Define new directory layout
├─ Task 10: Implement migration
├─ Task 11: Environment variables
└─ Task 12: Storage tests
        ↓  (required by Phase 3 & 4)
PHASE 3: UPDATE SYSTEM (Tasks 13–20)
├─ Task 13: Update-mode options in config.json
├─ Task 14: Version tracking functions
├─ Task 15: check_for_updates()
├─ Task 16: download_and_build_version()
├─ Task 17: Smoke tests
├─ Task 18: activate_version() & cleanup
├─ Task 19: Integrate update logic
└─ Task 20: Test update system
        ↓  (required by Phase 4)
PHASE 4: SNAPSHOTS (Tasks 21–25)
├─ Task 21: Backup config in config.json
├─ Task 22: is_snapshot_restore()
├─ Task 23: Cache reuse after restore
├─ Task 24: Snapshot marker
└─ Task 25: Snapshot tests
        ↓  (optional parallel with Phase 5)
PHASE 5: HA INTEGRATION (Tasks 26–31)
├─ Task 26: Ingress config
├─ Task 27: Panel config
├─ Task 28: HA API enablement
├─ Task 29: send_ha_notification()
├─ Task 30: Notification integration
└─ Task 31: HA integration tests
        ↓
PHASE 6: DOCUMENTATION (Tasks 32–37)
├─ Task 32: INSTALLATION.md
├─ Task 33: CONFIGURATION.md
├─ Task 34: TROUBLESHOOTING.md
├─ Task 35: README.md
├─ Task 36: CHANGELOG.md
└─ Task 37: Add-on docs
        ↓
PHASE 7: TESTING (Tasks 38–42)
├─ Task 38: Fresh install test
├─ Task 39: Upgrade test
├─ Task 40: Update-mode tests
├─ Task 41: Snapshot test
└─ Task 42: Rollback test
        ↓
RELEASE (Tasks 43–45)
├─ Task 43: Bump version to 1.0.0
├─ Task 44: Create git tag
└─ Task 45: GitHub release
```

---

## Critical blockers

### Absolute blockers
1. **Task 5 (Bun/TypeScript fix)** – if builds don’t work, nothing else matters
2. **Task 8 (multi-arch test)** – must be working before doing storage work
3. **Task 12 (storage test)** – blocks update system & snapshot work
4. **Task 20 (update system test)** – blocks snapshots & HA integration validation

### Important dependencies
- Task 19 (update logic) depends on tasks 13–18
- Task 30 (notifications) depends on task 19 (update logic must exist)
- Task 25 (snapshot tests) depends on task 20 (update system must be verified)

### Parallel opportunities
- Phases 4 & 5 can run in parallel **after** Phase 3 is verified
- Documentation can start while implementation is in progress
- The testing phase can be parallelized by feature area

---

## Suggested execution order (high-level)

1. **Critical path (sequential):**
   - 5 → 8 → 12 → 20 → 25

2. **Parallel where possible:**
   - Repo setup tasks (2–4) can be done in parallel
   - Start Phase 3 by parallelizing the “config/options” and “tracking functions” work
   - Phase 4 + Phase 5 in parallel once Phase 3 is proven stable
   - Documentation continuously as features stabilize

3. **Validation points:**
   - After Task 8: multi-arch builds succeed
   - After Task 12: migration + storage layout stable
   - After Task 20: update system safe (no broken activations)
   - After Task 25: snapshots/restores safe
   - After Phase 7: all tests green

---

## Next steps

- Continue with **Phase 6 (Documentation)** tasks 32–37
- Then complete **Phase 7 (Testing)** tasks 38–42
- Finish with **Release** tasks 43–45
