# Cloud — Attacks RDP NLA fix (1.4.88)

> Paste / implement against this note. Client SoT: [`../features/threat-engine.md`](../features/threat-engine.md).  
> Agent pin: **≥ 4.9.121**. Contract: **1.4.88**.

## Problem (lab 2026-09-09)

External wrong-password `mstsc` → Ninety (`194.5.236.32`) did **not** create Attacks **`RDP`** rows.  
SMB `net use` wrong password → **`NETWORK:445`** worked (`source=eventlog`).

Root cause is **client classification**, not cloud remap:

1. NLA fail → Security **4625 LogonType 3**, often **NTLM + NtLmSsp**, **no Event 1149** (1149 is success-path).
2. Client treated NtLmSsp as SMB → `NETWORK:445`, or dropped if RDP-classed without enabled RDP rule.
3. Without **RdpCoreTS Event 131** IP hints, fail spray never got an RDP mark.

## Cloud MUST (do not “fix” by remapping)

| # | Instruction |
|---|-------------|
| 1 | Keep `normalize_service` = **client string as-is**. Do **not** map `NETWORK`→`RDP` or reverse on ingest. |
| 2 | Fleet / update banner: recommend agent **≥ 4.9.121** for Attacks RDP NLA accuracy. |
| 3 | Dashboard Attacks: show enrich fields when present (`logon_type`, `auth_package`, `logon_process`, `workstation`, `source=eventlog`, real `port`). |
| 4 | Confirm Ninety (and fleet) has **`protection.block_rules` RDP enabled**. Auth-fail `/api/attack` is gated on an enabled rule covering that service — RDP disabled ⇒ silence even when classified correctly. |
| 5 | Bait vs real: FTP/MySQL/SSH **`source=honeypot`** only if tunnel **started**. Stopped baits + open real ports need AuthLogWatcher / EventLog — not empty Attacks. |
| 6 | Relocated RDP (e.g. **43389**): expect Attacks **`RDP` / `43389`**, not `0` and not `445`. |
| 7 | Optional UX: filter chips already have RDP vs Network — after pin, re-run external mstsc lab and tick **C-ATK-1**. |

## Lab acceptance (operator)

1. Pin Ninety to **≥4.9.121**, restart agent.
2. From external IP: `mstsc` wrong password → Attacks: **`RDP`**, real listen port, user, `<failed_logon>`, `source=eventlog`.
3. Same host: `net use \\host\IPC$` wrong password → **`NETWORK` / `445`** (separate row).
4. Do **not** expect honeypot FTP/MySQL rows unless Servis Tuzakları tunnels are **started**.

## Related

- Notify / digest: [`ATTACKS_NOTIFY_DUAL_CHANNEL.md`](./ATTACKS_NOTIFY_DUAL_CHANNEL.md)
- Checklist: [`CLOUD_CHECKLIST.md`](./CLOUD_CHECKLIST.md) · [`../features/CLIENT_CHECKLIST.md`](../features/CLIENT_CHECKLIST.md) C-ATK-*
