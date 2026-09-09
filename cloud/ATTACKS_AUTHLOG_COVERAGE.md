# Cloud — Attacks AuthLog coverage (1.4.89)

> Paste to cloud agent. Contract **≥ 1.4.89** · Client pin **≥ 4.9.122**.

## What changed (client)

AuthLogWatcher now tails:

| Service | Log | Match |
|---------|-----|-------|
| **MYSQL** | MySQL/MariaDB / XAMPP / Laragon `*.err` (+ probe) | `Access denied for user 'u'@'ip'` |
| **FTP** | IIS `FTPSVC*` W3C (+ probe) | `sc-status` **530** |
| **MSSQL** | `ERRORLOG*` (+ Application **18456** EventLog) | `Login failed … [CLIENT: ip]` |

Loopback / `localhost` sources are **not** reported. Health snapshot may include
`auth_log_health` (`mysql_count` / `ftp_count` / `mssql_count`, emits).

Lab inject (on agent host): `tools/sim_authlog_inject.py` →
`%ProgramData%\Asteria\authlog_probe\`.

## Cloud MUST

1. **Pin** recommend agent **≥ 4.9.122** for FTP/MYSQL/MSSQL file-tail Attacks.
2. **No** service remap — trust client `service` + `port`.
3. Confirm `protection.block_rules` enabled for **FTP**, **MYSQL**, **MSSQL** (and RDP/SSH as needed).
4. Dual channel unchanged: `source=honeypot` only when bait tunnels **started**; AuthLog / EventLog → `eventlog` (or channel-specific).
5. Lab: after inject or real spray, expect Attacks rows `FTP:21` / `MYSQL:3306` / `MSSQL:1433` (or relocated listen ports).

## Do not

- Require bait tunnels for real-service auth fails.
- Treat empty AuthLog path counts as product failure when those services are not installed (probe/sim still valid).
