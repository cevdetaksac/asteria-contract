# Defense policy — single contract

> **SoT** **≥ 1.4.61**. Pointers: [`../cloud/DEFENSE_POLICY.md`](../cloud/DEFENSE_POLICY.md),
> [`../agent/defense-policy-client.md`](../agent/defense-policy-client.md)

Tiers: observe → balanced → paranoid (+ isolate armed separately).
Fresh install defaults **observe**. Auto-promote observe→balanced (default 3 days, lockable).
Never auto-open paranoid/isolate. Tamper → LKG/observe, never escalate.

Hard-reject `auto_isolate_network` on observe/balanced. `isolate_host` only if paranoid+armed.
Canary/VSS/critical process gated by matrix (`alert_only` | `suspend_process` | `kill_quarantine`).
`allow_process` / `isolate_host` confirm-gated on cloud.
