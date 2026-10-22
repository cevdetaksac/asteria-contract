# Containment — process / host / playbooks

> **SoT** **≥ 1.4.106**. Client **≥ 4.9.145**. Admin result shape: [`server-management.md`](./server-management.md).

Operator-driven breach containment. No covert shell. Every mutating command uses
`{ok, success, code, message, steps[], data}` and (where reversible) returns `undo_id`.

## Process containment (P0)

| command_type | params | confirm | Notes |
|---|---|---|---|
| `suspend_process` | `pid`, `expected_image`, `process_start_time` | yes | Thread suspend only |
| `resume_process` | same identity | no | |
| `contain_process` | identity + `net_block?` `persistence?` `suspend?` `ifeo?` `dry_run?` | yes (dry_run exempt) | Suspend + per-program outbound WF `AR-CONTAIN-*` + disable linked service/schtasks/Run/RunOnce → `undo_id` |
| `release_process` | `undo_id` | no | Undoes contain (resume + delete WF rule + restore persistence + clear IFEO if set) |
| `block_process` | `process_name` / `path` | yes | IFEO Debugger stub |
| `unblock_process` | image name | no | Clears IFEO |
| `kill_process` | exact identity | yes | Existing admin v2 |

Asteria cloud / agent traffic is never targeted by per-program contain rules
(rules are program-path outbound blocks only).

## Accounts / services (P0)

| command_type | params | confirm |
|---|---|---|
| `disable_service` | `service_name` / `name`, `stop_first?` | yes → `undo_id` |
| `delete_user` | `username` | yes (permanent) |
| `enable_account` / `disable_account` / `reset_password` | SAM username (+ password) | existing |

Restore a disabled service: `disable_service` with `{undo_id, restore:true}` or
`sc config` via agent `enable_service_undo` path.

## Host isolate (P1)

| command_type | params | confirm |
|---|---|---|
| `isolate_host` | `management_ip?`, `dead_man_minutes` (5–120, default 15), `dns_servers?`, `dry_run?` | **strong yes** (dry_run exempt) |
| `lift_isolation` | `undo_id?` (defaults to active) | yes |

Allowlist: resolved Asteria cloud IPs + DNS (1.1.1.1/8.8.8.8/9.9.9.9) + optional
`management_ip`. Catch-all block In/Out. **Dead-man:** if agent cannot reach the
cloud for `dead_man_minutes`, auto `lift_isolation`. Resumed after agent restart.

## Playbooks (P2)

| command_type | params | confirm |
|---|---|---|
| `list_playbooks` | — | no |
| `run_playbook` | `playbook` ∈ `rdp_attack` | `compromised_admin` | `malicious_process`, scenario params, `dry_run?` | yes (dry_run exempt) |
| `undo_playbook` | `undo_id` | no |

Packs compose existing commands; each step lands in command history; one
`undo_id` reverses applied steps (best-effort, reverse order).

## UI (Control / Server mgmt)

Process row: **Dondur** / **Net kes** / **Öldür** / **Kalıcı engelle** / **Geri al**.
Users: delete_user. Services: disable_service. Isolate: strong confirm copy.
