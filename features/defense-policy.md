# Defense policy — single contract

> **SoT** **≥ 1.4.65**. Tiers: observe → balanced → paranoid (+ isolate armed separately).

Fresh install defaults **observe**. Auto-promote observe→balanced (default 3 days, lockable).
Never auto-open paranoid/isolate. Tamper → LKG/observe, never escalate.

Hard-reject `auto_isolate_network` on observe/balanced. `isolate_host` only if paranoid+armed.
Canary/VSS/critical process gated by matrix (`alert_only` | `suspend_process` | `kill_quarantine`).
`allow_process` / `isolate_host` confirm-gated on cloud.

## Cloud invariants

1. Observe / Balanced default: no network isolation.
2. One canary / one VSS intent must not queue `isolate_host`.
3. `under_attack` only on verified red events (not soft surface info).
4. N alerts ≠ auto escalate / isolate.
5. Broken policy signature → client LKG/observe; cloud does not “go aggressive”.
6. Auto-promote is **only** observe → balanced.
