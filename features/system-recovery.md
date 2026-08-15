# System recovery — single contract

> **SoT** **≥ 1.4.62**. Pointer: [`../agent/system-recovery.md`](../agent/system-recovery.md)

Allowlist snapshot of TaskMgr/Regedit/CMD policy, critical services, firewall
profile. Drift watch + `system_recovery_restore` (dry_run exempt from confirm).
No full registry dump. HKCU via `HKEY_USERS\<interactive-SID>` from Session-0.
