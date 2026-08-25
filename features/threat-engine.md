# Local threat engine — single contract

> **SoT** **≥ 1.4.75**. Cloud IoC bundle is [`threat-intel.md`](./threat-intel.md) (separate).
> Client **≥ 4.9.108**: real-port EventLog fails honor `protection.block_rules` even when honeypot is off;
> 4625 NLA → **RDP:3389**; SMB/null → **Network:445** (`port:0` yasak when known).

EventLog / score / `AR-BLOCK` auto-response. Bare RDP success is not
score 100 / auto-block. Whitelist never blocked. Alert hygiene: VSS list is not
ransomware; canary self-touch suppressed.

## Real-port auth (honeypot optional)

1. Windows Security `4625` / TerminalServices / MSSQL fail events feed
   `ThreatEngine` with `target_service` (see **4625 classification** below).
2. Enabled `protection.block_rules` (or local defaults) count **per service**
   and block at threshold — independent of bait honeypot listen state.
3. Matching fails also POST `/api/attack` (password placeholder
   `<failed_logon>`) so dashboard Attacks / notification counters work without
   honeypot. Bait captures remain a separate channel
   ([`agent/attacks-and-services.md`](../agent/attacks-and-services.md)).

## 4625 classification (MUST)

RDP with NLA/CredSSP writes Security **4624/4625 as LogonType 3** (Network),
not type 10. Mis-labeling those as `Network` fills the wrong Attacks bucket and
never hits the RDP fail-3 rule.

| Signal | `target_service` | `target_port` |
|--------|------------------|---------------|
| LogonType **10** | `RDP` | **3389** (or real TermService listen) |
| LogonType **3** + RDP/NLA mark | `RDP` | **3389** (or real listen) |
| LogonType **3** + SMB / null-session | `Network` (or `SMB`) | **445** — **`port:0` forbidden** when known |

**RDP/NLA marks** (any one is enough):

- `AuthenticationPackage` **Negotiate** and/or `LogonProcess` **User32** (or Advapi NLA path)
- Recent **TerminalServices Event 1149** for the same source IP (TTL ~3 min)
- Explicit TerminalServices / process-term context on the event

**SMB / Network marks:**

- `LogonProcess` **NtLmSsp** (typical 445 / null-session / SMB auth)
- Type-3 with NTLM + no RDP marks above → `Network`, port **445**

Do **not** treat every LogonType 3 as an attack spam row:

- Empty / `-` / `ANONYMOUS LOGON` → keep out of Attacks flood (or score low); do **not**
  burn RDP threshold-3 on them.
- Admin / named-user spray with RDP marks → count toward **RDP** rule (threshold 3).

## `/api/attack` enrichment (EventLog path)

When reporting real-port fails, include at least (when present on 4625):

`logon_type`, `auth_package`, `logon_process`, `status` / `substatus`,
`workstation`, real `port`, `source: eventlog`.

Dashboard “Network mi RDP mi?” depends on **client** `service` + these fields.
Cloud `normalize_service` writes the client string as-is — **RDP↔Network remap is client SoT**.

## Acceptance (lab)

- [ ] NLA-on host, external RDP fail → Attacks: **`RDP`**, **`port=3389`**, password `<failed_logon>` (not `NETWORK`/`port:0`)
- [ ] Pure 445/SMB fail → **`Network`** (or SMB), **`port=445`**
- [ ] Bait RDP started + credential → **separate** bait RDP row (`source` ≠ eventlog-only flood)
- [ ] Under RDP NLA spray, NETWORK must not be the single catch-all bucket
