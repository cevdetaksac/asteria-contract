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

## Incident response (1.4.91)

| Action | command_type | params | UI |
|---|---|---|---|
| Contain user | `contain_user` | `username`, `new_password` (>=8), `disable?`, `session_id?`, `confirm_name` | Users row action (shield) |
| Disable all users | `disable_all_users` | `logoff:true`, `exclude[]`, `confirm_name` | Users header + IR card |
| Emergency lockdown | `enable_lockdown` | `management_ip`, `duration_minutes` (5-1440), `confirm_name` | Users header + IR card |

`confirm_name` must equal the server name (case-insensitive); cloud strips it before queueing.
All three require `confirm=true` and are written to the command audit log.

## Incident response sessions & full lockdown (1.4.92)

| Action | command_type | params | Notes |
|---|---|---|---|
| Log off one session | `logoff_session` | `session_id`, `dry_run?` | WTSLogoffSession; skips session 0 / agent |
| Disconnect session | `disconnect_session` | `session_id`, `dry_run?` | WTSDisconnectSession |
| Log off everyone | `logoff_all_sessions` | `exclude_session_ids?`, `exclude_users?`, `include_console?`, `dry_run?` | Per-session actions[] |
| Emergency lockdown | `enable_lockdown` / `emergency_lockdown` | `management_ip`, `duration_minutes`, `logoff_all?`, `block_rdp?`, `disable_non_admin?`, `keep_admins?`, `dry_run?`, `confirm_name` | RDP ports from registry+3389; undo_id |
| Undo lockdown | `emergency_lockdown_undo` | `lockdown_id` | Re-enable users + delete ASTERIA_IR_RDP_* rules |

Result: `{ok, success, actions:[{type,target,ok,error}], undo_id?}`.
