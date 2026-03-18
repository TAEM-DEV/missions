---
TAEM Post-Remediation Review: ry-ops/git-steer
---
**Mission:** Post-remediation review of git-steer after Cycle 1
---
**Date:** 2026-03-18
---
**Target:** https://github.com/ry-ops/git-steer (post PR #37)
---
**Prior Review:** [git-steer.md](git-steer.md) — Verdict: HOLD
---
**Remediation Plan:** [git-steer-remediation-cycle-1.md](git-steer-remediation-cycle-1.md)
---
**Mission ID:** MSN-8f169d323a193d51
---
**Verdict:** **GO** — all Cycle 1 blocking findings resolved

---

## Remediation Verification

### SECINSP — Was NO-GO, Now GO

| Finding | Status | Verification |
|---------|--------|-------------|
| SEC-001 (token in `process.env`) | **RESOLVED** | `gateway.ts` now saves original env values, sets them for `createApp()`, then deletes/restores. Token no longer persists in global env. |
| SEC-002 (token as public property) | **RESOLVED** | `FabricGitHubAdapter.token` replaced with `headers()` method. Raw token never exposed through interface. |
| SEC-003 (token in Authorization headers) | **RESOLVED** | All 10 instances of `Authorization: token ${github.token}` in `git.ts` replaced with `github.headers()`. Zero raw token references remain. |
| SEC-010 (unguarded destructive tools) | **RESOLVED** | `permissions.ts` defines `DESTRUCTIVE_TOOLS` (6 tools) and `DRY_RUN_DEFAULT_TOOLS` (4 tools). Guard in `CallToolRequestSchema` handler requires `confirm: "CONFIRM_<TOOL_NAME>"` for destructive ops. Sweep tools default to `dry_run: true`. |

**SECINSP Signal: GO** on first pass (Phase 04, latency 16,750ms via Ollama).

---

### TRC — Was NO-GO, Now GO

| Check | Before | After |
|-------|--------|-------|
| Test files | 5 | 8 |
| Total tests | 35 | 42 |
| Permissions tests | 0 | 7 (destructive tool + dry-run classification) |
| Adapter tests | 0 | 2 (headers method, no raw token) |
| Gateway env tests | 0 | 2 (save/restore/delete behavior) |
| Coverage config | None | vitest v8 provider, 60/60/50/60 thresholds |
| Coverage reporting | None | text + lcov reporters |

**TRC Signal: GO**

---

### PRB — Was NO-GO (2/3), Advisory

| Sub-Agent | Before | After | Notes |
|-----------|--------|-------|-------|
| PRB-SKP | NO-GO | NO-GO | Flagging empty step plan — expected for review-only missions |
| PRB-COR | NO-GO | NO-GO | Same — "steps and order missing" is correct for a plan with 0 implementation steps |
| PRB-ADR | NO-GO | NO-GO | Same — no steps to evaluate against ADR-001 |

**PRB Assessment:** The PRB NO-GO votes are structurally correct — the step plan IS empty because this is a review mission, not an implementation mission. The PRB agents are doing their job (flagging incomplete plans). This is a TAEM protocol observation, not a git-steer code issue:

> Review-only missions produce empty step plans. PRB controllers evaluate step plans. An empty plan will always fail PRB's completeness check. TAEM should consider a mission type flag (`review` vs `implement`) that adjusts PRB's expectations.

The original PRB NO-GOs cited:
- **PRB-SKP**: Unguarded destructive tools, dual Octokit → **Both resolved** (permissions guard, single Octokit)
- **PRB-ADR**: Zero-footprint violations → **Resolved** (conditional tool registration)

---

### Additional Findings — Resolved

| Finding | Status | Verification |
|---------|--------|-------------|
| ARCH-001/002 (god files) | **RESOLVED** | `server.ts` reduced from 3,251 to 612 lines. 10 per-domain tool modules in `src/mcp/tools/`. |
| CDS-001 (dual Octokit) | **RESOLVED** | `fabric/app.ts` now accepts `{ octokit: Octokit }` — no more `createOctokit()`. Single throttled client. |
| ARCH-005 (hardcoded owner) | Deferred to Cycle 2 | |
| ARCH-006 (`dist/` in source) | Deferred to Cycle 2 | |

---

## Phase-by-Phase Results

| Phase | Controllers | Signals |
|-------|------------|---------|
| 00 — Pad Check | GC, DPS, EECOM | All **GO** |
| 01 — Corpus Ingestion | NAV | **GO** (dispatched, integration-map received) |
| 02 — Architectural Survey | ARCH, CDS, PCO | All **GO** |
| 03 — Plan Formulation | INCO | **GO** (0 steps — review mission) |
| 04 — Pre-Code Inspection | SECINSP, TRC, PRB | SECINSP **GO**, TRC **GO**, PRB **NO-GO** (empty plan) |
| 05 — CAPCOM Output | CAPCOM | **RELAY** — LANDED |
| 06 — PAO Dispatch | PAO | NO-GO (workflow missing — TAEM infra gap, not git-steer) |

---

## Gate Decision

```
Phase 04 Gate: CONDITIONAL GO
  SECINSP: GO     ← was NO-GO, now cleared
  TRC:     GO     ← was NO-GO, now cleared
  PRB:     NO-GO  ← structural (empty step plan), not code quality

Original blocking findings: ALL RESOLVED
  SEC-001: RESOLVED (env cleanup)
  SEC-002: RESOLVED (headers method)
  SEC-003: RESOLVED (no raw token refs)
  SEC-010: RESOLVED (permission guard)
  CDS-001: RESOLVED (single Octokit)
  ARCH-001/002: RESOLVED (10 modules)
  TRC: RESOLVED (42 tests, coverage config)

Recommendation: ADVANCE to Cycle 2 (non-blocking items)
```

---

## Cycle 2 Candidates (Non-Blocking)

These were deferred per ADR-003 C-003-004. None are gate-blocking:

| Priority | ID | Description |
|----------|----|-------------|
| P2 | SEC-004 | Validate and allowlist Slack webhook URLs |
| P2 | SEC-005 | Remove `--force` from `npm audit fix` in CI |
| P2 | SEC-011 | Quote `${{ matrix.target.* }}` in workflow shell commands |
| P2 | DPS | Add Zod schemas for MCP tool inputs |
| P2 | CDS-003 | Document state write conflict model |
| P2 | ARCH-005 | Derive dashboard owner/repo from state |
| P2 | ARCH-006 | Remove `dist/` from source control |

---

## TAEM Infrastructure Observations

During this review cycle, three TAEM kernel issues were found and fixed:

1. **NAV dispatch path** — `controllers.yaml` used `.github/workflows/nav.yml` but GitHub's API expects just `nav.yml`. Fixed in TAEM-DEV/taem@8d1e0c9.

2. **Inference controllers not wired** — `realControllerFactory` returned `nil` for SECINSP, PRB-SKP, PRB-COR, PRB-ADR. They were never running. Fixed in TAEM-DEV/taem@39223c4.

3. **Ollama timeout too aggressive** — 45s default insufficient for llama3.2 on k3s workers handling concurrent requests. Bumped to 120s default, 180s per-controller. Fixed in TAEM-DEV/taem@d2fc73d.

4. **PAO workflow missing** — Phase 06 escalates because `controllers/pao.yml` doesn't exist in mc-state. PAO is `required: false` but the gate still processes it during remediation. Recommend: skip PAO dispatch entirely when workflow doesn't exist.

5. **Review missions and PRB** — Empty step plans are correct for review-only missions but always fail PRB completeness checks. Recommend: add mission type flag so PRB adjusts expectations.

---

## What Changed in git-steer (PR #37)

22 files changed, 3,533 insertions, 2,674 deletions.

| Category | Files |
|----------|-------|
| Security | `adapter.ts`, `git.ts`, `gateway.ts`, `permissions.ts` |
| Architecture | `server.ts` + 10 new `tools/*.ts` modules |
| Client | `client.ts` (getOctokit), `app.ts` (accept Octokit) |
| Tests | `permissions.test.ts`, `adapter.test.ts`, `gateway-env.test.ts` |
| Config | `vitest.config.ts` (coverage) |

---

*Generated by TAEM kernel review protocol. Post-remediation verification for MSN-8f169d323a193d51.*
*Controllers: GC, DPS, EECOM, NAV, FAO, ARCH, CDS, PCO, INCO, SECINSP, TRC, PRB-SKP, PRB-COR, PRB-ADR, CAPCOM, PAO.*
