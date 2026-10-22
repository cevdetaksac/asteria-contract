# Ağ & Portlar (1.4.107)

Single Control page: **Kontrol → Ağ & Portlar** (`/dashboard/network`).
Legacy `/dashboard/ports` and `/dashboard/server/firewall` redirect here.

## Agent commands
| Command | Purpose |
|---|---|
| `list_firewall` | Inventory; every rule has stable `id`/`guid` |
| `firewall_rule` | `op=enable\|disable\|delete\|add\|edit` — ops by `id` or `name`; edit uses Set-NetFirewallRule / AddressFilter; every mutate returns `undo_id` |
| `firewall_undo` | Restore via `undo_id` |
| `port_expose` | `mode=all\|ips\|close` for a local TCP port; snapshot+undo; dead-man on risky RDP changes |
| `sql_auth_diag` | Read-only: SQL login mode + Application 18456 count (24h) + ERRORLOG lines |

## Self-lockout guard
- Refuse outbound Block on 443 / asteria.run
- Refuse disable/delete of RDP allow; refuse edit that drops `management_ip`
- Refuse `port_expose` close on RDP listen port unless `force` + typed confirm
- Dead-man (≈5 min): if cloud unreachable after risky change → auto undo

## Exposure column
`open` (Herkese açık) / `restricted` (Kısıtlı) / `closed` (Kapalı) / `local_only` (Sadece yerel).
Risky highlight: 1433 / 3306 / 3389 open to Any.

## UI note
**Net kes** (`contain_process`) blocks outbound for the **entire program path** (all PIDs of that exe).
