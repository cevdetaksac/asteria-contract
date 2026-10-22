# Brute-force SQL fixes (1.4.107)

Client floor **≥ 4.9.146**. Honeypot on/off irrelevant — EventLog + ERRORLOG feed rules.

## MSSQL 18456
- Unnamed `<Data>` → `ParamN`; login from `Login failed for user '…'` / Param1; IP from `[CLIENT: …]`
- Empty-username filter **not** applied to SQL auth events

## Counting
- AuthLogWatcher emits **every** fail (no 10s dedup into the engine)
- ThreatEngine 15s dedup only for Attacks list / e-mail; carries `attempts`
- Cloud `Attack.attempts` weighted in threshold + catch-up
- MSSQL ERRORLOG ↔ Application 18456 cross-channel dedup (3s)

## Rules
- Long-window defaults: MSSQL/MYSQL **5 / 24h**
- Cloud catch-up floor for SQL-only rules: `max(threshold, 5)` (was `max(3×threshold, 20)`)
- `/24` spray: ≥3 distinct IPs in same public /24 over 24h → block the /24
- Optional `protection.sql_privileged_first_fail` (sa/root, internet-exposed only)

## Labels / health
- Sources: `mysql_log` / `mssql_errorlog` / `ftp_log` / `eventlog`
- MySQL hostnames resolved to IP (cached)
- `system_health.auth_log_health` stored on cloud
