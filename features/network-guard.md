# Network Guard — single contract

> **SoT** **≥ 1.4.65**. Detection is **always alert-only**.

Cloud cannot enable `auto_contain` / `auto_kill` / `auto_restore` as silent kill.
Suspend only via confirmed `suspend_process` (exact pid).

Golden baseline: `network_snapshot` / `list_network_baseline` / `network_diff`.
`network_restore` mutating = confirm; `dry_run:true` is plan-only (no confirm).
Soft `network_surface_changed` on additive adapter/DHCP — never auto-disable.
Maintenance chip: pause / snapshot / resume (`network_maintenance_*`).
