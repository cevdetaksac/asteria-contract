# Server management — single contract

> **SoT** **≥ 1.4.65**. Process evidence: [`process-inspect.md`](./process-inspect.md)

Three dashboard pages: users, processes, services. Agent returns **full inventory**
on refresh and executes catalog mutates. No covert shell.

Users: local SAM + WTS; disable not delete; `include_disabled`; confirm on
create / remote_logon / reboot. Processes: lean health + `inspect_process` (no confirm).
Services: pywin32 SCM; protected services refuse stop.

## C-USER (dashboard)

| ID | Cloud |
|----|--------|
| **C-USER-1** | `list_local_users` always `include_disabled: true` |
| **C-USER-2** | Enabled + Disabled rows together (filter optional). Wire `status` = SAM `enabled`/`disabled` (agent ≥4.9.114) — not WTS `Active`. Live session only via `has_session` + `session_id` + `session_status` ∈ Active/Connected. |
| **C-USER-3** | Enable / Disable toggle (`enable_account` / `disable_account`) |
| **C-USER-4** | After mutate, refresh inventory |
| **C-USER-5** | Cache keeps `enabled:false` rows |
| **C-USER-6** | Confirm before disable/enable; `PROTECTED_ACCOUNT` is a visible error |
| **C-USER-7** | Enabled / disabled counts in the header (`counts.enabled`; `counts.active` = legacy alias) |
