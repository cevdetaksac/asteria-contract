# Promotion gates — observe → enforce (single SoT)

> **Contract VERSION:** **1.4.37**  
> Status: **Normative**  
> Related: [`FLEET_CANARY.md`](./FLEET_CANARY.md) · [`../api/10-offline-urgent-queue.md`](../api/10-offline-urgent-queue.md) · [`../features/defense-policy.md`](../features/defense-policy.md) · [`ZERO_TRUST_STATUS.md`](./ZERO_TRUST_STATUS.md) · [`../api/12-command-envelope-v2.md`](../api/12-command-envelope-v2.md)

This file answers: **when may an observe-mode feature move to enforce / fleet-on?**  
Scattered checklists elsewhere point here; do not invent parallel criteria.

---

## Global rules

1. **Fail-closed.** Missing gate = stay observe / flag off / auto-actions cleared.
2. **Canary first.** Any automation that can brick IR or isolate a host MUST pass
   [`FLEET_CANARY.md`](./FLEET_CANARY.md) before fleet percent > 0 for that gate.
3. **Separate contract bump.** Enforce / floor change ships in an **explicit**
   VERSION release — never silent.
4. **Regression harness green** on the feature’s named tests before canary opens.
5. **Rollback path** documented (env flip or per-client override) before open.

---

## Gate table

| Feature | Observe today | Enforce / fleet-on requires ALL of |
|---------|---------------|-------------------------------------|
| **v1 HMAC command signing** (`asteria-chp-v1`) | Soft-allow missing sig | Fleet `coverage_percent ≥ 99.5` for **≥7 days**; `signature_generation_failed == 0`; client ≥ floor verifying Asteria context; explicit contract note flipping soft-allow → reject |
| **Command envelope v2** | Schema normative (`api/12`); wire emit **off** | Gates in `api/12` §Emit gates; dual-read/write; pilot hosts; **separate** VERSION for emit then again for enforce |
| **Offline urgent queue** | Cloud endpoints live; client flag default **off** | One-host live pilot pass (`api/10`); canary gate `offline_urgent_queue` open ≥**7 days** at ≤5% with zero duplicate-incident regressions; then staged %; fleet default on only in later VERSION |
| **Silent-hours auto-actions** | Config may store flags; cloud **clears** out-of-canary | Anti-brick C-BRICK satisfied; canary `silent_hours_auto_actions` ≥7d; undo-mail E2E green; no unlinked auto-disable incidents |
| **Network Guard auto-contain / auto-kill** | Policy may request; cloud clears out-of-canary | Soft-inform + golden baseline stable; canary `network_guard_auto_contain` ≥7d; no false-contain on DHCP/VPN churn lab |
| **Defense `isolate_armed`** | Observe→balanced auto-promote only | Never auto-arm; canary `defense_isolate_armed` + operator confirm path; paranoid preset only |
| **Defense observe→balanced** | Auto-promote after N days | Already normative (`DEFENSE_POLICY`); never auto→paranoid / never auto-arm isolate |
| **Legacy dual-brand host / HMAC verify** | Dual accept during cutover | Sunset criteria in [`PRODUCT_BRANDING.md`](./PRODUCT_BRANDING.md) §Sunset |

---

## Metrics sources (cloud)

| Metric | Where |
|--------|--------|
| Per-client + fleet signing coverage | `GET /api/security-resilience/status` → `command_signing` (+ `.fleet`) |
| Canary membership / gates | `GET /api/threats/config` → `fleet_rollout` |
| Offline queue health | `POST /api/health/report` → `offline_urgent_queue` |

---

## Operator checklist (before raising `ASTERIA_FLEET_CANARY_PERCENT`)

- [ ] Feature env master ON (`ASTERIA_CANARY_*`) for **only** the feature under test  
- [ ] Percent ≤ 5 for first window; `observe_days_recommended` (≥7) calendar booked  
- [ ] Force-include lab IDs only via `ASTERIA_CANARY_FORCE_IDS`  
- [ ] Dashboard watch: signing failures, false contains, brick/undo mail volume  
- [ ] Rollback: set percent=0 and/or feature env=0 (cloud clears auto flags on next config poll)

---

## Explicit non-gates

- “Dashboard looks fine for an afternoon” ≠ promote.  
- Coverage on a single lab host ≠ fleet coverage.  
- Design-complete ≠ wire emit (`api/12`).
