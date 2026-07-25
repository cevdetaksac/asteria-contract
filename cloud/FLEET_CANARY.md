# Fleet canary / staged rollout

> **Contract VERSION:** **1.4.37**  
> Status: **Normative** (cloud live; client must honor gates)  
> Min client for **AND** defense-in-depth: **≥ 4.9.37**  
> Cloud still fail-closes auto flags for older clients via config mutation.

Automation that can mislead IR or brick a host (silent-hours auto-disable,
Network Guard auto-contain, offline urgent queue enablement, isolate arming)
MUST NOT go fleet-wide in one flip. This doc is the wire + ops SoT.

Related: [`PROMOTION_GATES.md`](./PROMOTION_GATES.md) · [`../agent/anti-brick-critical-actions.md`](../agent/anti-brick-critical-actions.md)

---

## Wire — `fleet_rollout` on threats/config

Emitted on every `GET` / effective `POST` `/api/threats/config`:

```json
{
  "fleet_rollout": {
    "schema": "fleet_rollout/1.0",
    "cohort_id": "canary-default",
    "percent": 0,
    "bucket": 42,
    "in_canary": false,
    "forced": false,
    "gates": {
      "silent_hours_auto_actions": false,
      "network_guard_auto_contain": false,
      "offline_urgent_queue": false,
      "defense_isolate_armed": false
    },
    "observe_days_recommended": 7,
    "message": "Automation canary — apply risky auto-actions only when gate=true AND local config enables the feature"
  }
}
```

| Field | Meaning |
|-------|---------|
| `bucket` | Stable `0..99` from `sha256("asteria-canary-v1:{client_id}")` |
| `percent` | Ops canary width (`ASTERIA_FLEET_CANARY_PERCENT`) |
| `in_canary` | `forced` OR `bucket < percent` (when percent > 0) |
| `gates.*` | Feature allowed **for this host** (canary ∩ feature master env) |

### Cloud fail-closed mutation

When a gate is `false`, cloud **clears** the effective payload even if DB/config
stored `true`:

| Gate false | Cleared fields |
|------------|----------------|
| `silent_hours_auto_actions` | `silent_hours.auto_block_ip`, `auto_logoff`, `auto_disable_account` |
| `network_guard_auto_contain` | `protection.network_guard.auto_contain`, `auto_kill`, `auto_restore` |
| `defense_isolate_armed` | top-level + nested `isolate_armed` → `false` |

`enabled` / observe / alert-only knobs are **not** cleared — only risky auto.

---

## Client MUST (C-CANARY-*)

| ID | Rule |
|----|------|
| **C-CANARY-1** | Read `fleet_rollout.gates` on every config apply |
| **C-CANARY-2** | Treat auto-action as allowed only if `gate==true` **AND** local/config enable is true |
| **C-CANARY-3** | If `fleet_rollout` missing/malformed → all four gates = **false** (fail-closed) |
| **C-CANARY-4** | Do not cache a previous `true` gate across process restart without a fresh config pull |
| **C-CANARY-5** | Health/report may echo `fleet_rollout.in_canary` + gate map (observe) |

`offline_urgent_queue` gate: client may enable spool/drain **only** when gate true
**and** `security.offline_urgent_queue` local flag true (still default off).

---

## Ops env

| Env | Default | Meaning |
|-----|---------|---------|
| `ASTERIA_FLEET_CANARY_PERCENT` | `0` | 0 = nobody in percent cohort |
| `ASTERIA_CANARY_FORCE_IDS` | empty | Comma client ids always in canary |
| `ASTERIA_CANARY_COHORT_ID` | `canary-default` | Label for audits |
| `ASTERIA_CANARY_SILENT_HOURS_AUTO` | `0` | Master for silent-hours auto gate |
| `ASTERIA_CANARY_NG_AUTO_CONTAIN` | `0` | Master for NG auto-contain/kill |
| `ASTERIA_CANARY_OFFLINE_URGENT` | `0` | Master for offline queue enable |
| `ASTERIA_CANARY_ISOLATE` | `0` | Master for isolate_armed pass-through |

Optional per-client override in `settings_json.fleet_rollout_override`:
`{"force_canary": true}` or `{"force_out": true}`.

---

## Staged recipe

1. Feature master env = 1; percent = **5**; force lab ids.  
2. Observe **≥7 days** (`observe_days_recommended`); regression suite green.  
3. Raise percent 5 → 25 → 50 → 100 only if [`PROMOTION_GATES.md`](./PROMOTION_GATES.md) row satisfied.  
4. Rollback = percent `0` and/or feature env `0` (next poll clears autos).

---

## Acceptance

### Cloud
- [x] `fleet_rollout` on threats/config  
- [x] Out-of-canary auto flags cleared in payload  
- [x] Default percent 0 / feature masters off  

### Client (≥ 4.9.37)
- [ ] C-CANARY-1…5  
- [ ] Unit: gate false → no auto-disable / no auto-contain even if config true
