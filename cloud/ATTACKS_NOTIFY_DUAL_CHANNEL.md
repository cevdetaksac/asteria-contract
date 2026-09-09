# Cloud instruction — Attacks dual-channel + NETWORK email digest

> Paste to cloud agent. Contract **≥ 1.4.88** · Client pin **≥ 4.9.121** (NLA RDP).
> SoT: `features/threat-engine.md` · `agent/attacks-and-services.md` ·
> [`ATTACKS_RDP_NLA_FIX.md`](./ATTACKS_RDP_NLA_FIX.md)

## Context (prod)

- Lab saw ~12k/day `NETWORK port=0` vs few `RDP:3389` while tunnels stopped.
- Client **4.9.108+** fixes classification + `port:0`; **4.9.109** dual-channel
  real-port (EventLog / OpenSSH / MySQL errlog / IIS FTP 530) **independent of bait**.
- **4.9.121**: RdpCoreTS **131** + NLA NtLmSsp heuristics — external mstsc fail → **RDP**, not `NETWORK:445`.
- Email spam is **cloud** `notification_rules` / prefs — client does not send mail.

## MUST — service email policy

| `service` | Instant email | Digest |
|-----------|---------------|--------|
| `RDP` / `MSSQL` / `MYSQL` / `SSH` / `FTP` | Yes (threshold rules, e.g. fail-3) | Optional summary |
| `NETWORK` / `SMB` | **No** | **Hourly or daily** digest only (count + top IPs/users/hosts) |
| Bait (`source=honeypot`) | Same as service above | — |
| EventLog (`source=eventlog`) | Same | — |

- Do **not** remap client `service` strings (`normalize_service` = as-is).
- Prefer enriched fields when present: `logon_type`, `auth_package`, `logon_process`,
  `status`/`substatus`, `workstation`, `port`, `source`.
- `NETWORK` = LogonType-3 SMB/null noise bucket — not “all attacks”.

## MUST — dual channel (dashboard Attacks)

1. Rows with `source=eventlog` (or mysql/ftp log) appear when honeypot tunnels are **stopped**.
2. Rows with `source=honeypot` appear only when bait listen is **started**.
3. Same host can show **both** (e.g. bait RDP:3389 + real RDP:43389) — intentional leak signal.
4. Pin / floor banners: recommend agent **≥ 4.9.121** (Attacks NLA); dual-channel floor **≥ 4.9.109**.
5. RDP `block_rules` must stay **enabled** or EventLog RDP fails never POST `/api/attack`.

## Acceptance

- [ ] NETWORK fails do not fire per-event mail; digest ≤1/hour (or daily)
- [ ] RDP/MSSQL/MYSQL/SSH/FTP fail still instant (or rule threshold)
- [ ] Attacks UI shows `source` + real `port` from client payload
- [ ] No cloud RDP↔NETWORK remap
- [ ] External NLA mstsc fail → **RDP** row (agent ≥4.9.121) — see [`ATTACKS_RDP_NLA_FIX.md`](./ATTACKS_RDP_NLA_FIX.md)
