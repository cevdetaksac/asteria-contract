# Cloud — Attacks AuthLog coverage (1.4.90)

> Paste to cloud agent. Contract **≥ 1.4.90** · Client pin **≥ 4.9.123**.

## What changed (client)

AuthLogWatcher tails:

| Service | Log | Match |
|---------|-----|-------|
| **MYSQL** | MySQL/MariaDB `*.err` via globs + **my.ini** `datadir`/`log-error` (+ probe) | `Access denied for user 'u'@'ip'` |
| **FTP** | IIS `FTPSVC*` W3C (+ probe) | `sc-status` **530** |
| **MSSQL** | `ERRORLOG*` via globs + **SQL instance registry** (+ probe) | `Login failed … [CLIENT: ip]` |

**MySQL 8 note:** default `log_error_verbosity=2` **does not** write Access denied
(Notes need **3**). Client **≥4.9.123** auto-sets verbosity=3 in `my.ini` on
install/boot (backup `.asteria.bak`, restart MySQL only if changed). Opt-out:
`ASTERIA_SKIP_MYSQL_AUTHLOG_PREP=1`. Health may still set `mysql_logging_hint`
when an err path exists but `mysql_access_denied_tail=0` (e.g. prep skipped /
custom datadir).

Loopback / `localhost` sources are **not** reported. Snapshot: `auth_log_health`.

Lab inject: `tools/sim_authlog_inject.py` → `%ProgramData%\Asteria\authlog_probe\`.

## Cloud MUST

1. **Pin** recommend agent **≥ 4.9.123**.
2. **No** service remap — trust client `service` + `port`.
3. Confirm `protection.block_rules` for **FTP**, **MYSQL**, **MSSQL**.
4. Dual channel unchanged: bait → `honeypot`; AuthLog/EventLog → `eventlog`.
5. Empty `mssql_count` is OK when SQL Server is not installed on the host.

## Do not

- Require bait tunnels for real-service auth fails.
- Treat missing MSSQL install as AuthLog product failure.
- Expect MYSQL Attacks while MySQL hides Notes (`log_error_verbosity<3`).
