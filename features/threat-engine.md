# Local threat engine — single contract

> **SoT** **≥ 1.4.76**. Cloud IoC bundle is [`threat-intel.md`](./threat-intel.md) (separate).
> Client **≥ 4.9.109**: dual-channel Attacks — **real listen ports always**, bait optional;
> OpenSSH Operational + MySQL error-log; relocate-aware ports.

EventLog / score / `AR-BLOCK` auto-response. Bare RDP success is not
score 100 / auto-block. Whitelist never blocked. Alert hygiene: VSS list is not
ransomware; canary self-touch suppressed.

## Dual channel (MUST)

Honeypot **on or off** must not gate real-port brute-force reporting.

| Channel | When | `source` | Port |
|---------|------|----------|------|
| **Real service** | Windows EventLog / OpenSSH Operational / MySQL error log | `eventlog` (or `mysql_error_log`) | **Actual listen** (e.g. RDP relocated **43389**) |
| **Bait honeypot** | Tunnel `started` + protocol credential | `honeypot` | Bait listen (often 3389/22/…) |

If bait is on **and** real RDP was relocated, both rows appear: bait spray on 3389 **and** hits on the hidden real port. That is intentional — operator sees the leak.

## Real-port auth sources

| Service | Capture (honeypot irrelevant) | Default / relocate port |
|---------|-------------------------------|-------------------------|
| **RDP** | Security `4625` (+ NLA marks) / TS 1149 | TermService registry / open_ports |
| **Network/SMB** | `4625` NtLmSsp | **445** |
| **MSSQL** | Application `18456` | 1433 or listen |
| **SSH** | `OpenSSH/Operational` Event 4 Failed/Invalid | 22 or listen |
| **MYSQL** | MySQL/MariaDB `*.err` “Access denied for user” | 3306 or listen |
| **FTP** | IIS FTP W3C logs (`FTPSVC*`, sc-status **530**) | 21 or listen |

Enabled `protection.block_rules` count **per service** and POST `/api/attack`
(`<failed_logon>`). Bait remains separate
([`agent/attacks-and-services.md`](../agent/attacks-and-services.md)).

## 4625 classification (MUST)

RDP with NLA/CredSSP writes Security **4624/4625 as LogonType 3** (Network),
not type 10. Mis-labeling those as `Network` fills the wrong Attacks bucket and
never hits the RDP fail-3 rule.

| Signal | `target_service` | `target_port` |
|--------|------------------|---------------|
| LogonType **10** | `RDP` | real TermService listen |
| LogonType **3** + RDP/NLA mark | `RDP` | real listen |
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

- [ ] NLA-on host, external RDP fail → Attacks: **`RDP`**, real port, `<failed_logon>` (not `NETWORK`/`port:0`)
- [ ] Pure 445/SMB fail → **`Network`**, **`port=445`**
- [ ] OpenSSH fail (sshd installed) → **`SSH`**, listen port, honeypot stopped OK
- [ ] MySQL “Access denied” in error log → **`MYSQL`**, honeypot stopped OK
- [ ] IIS FTP W3C **530** → **`FTP`**, honeypot stopped OK
- [ ] Bait RDP started + credential → **separate** bait row; real RDP fail still separate
- [ ] Relocated RDP (e.g. 43389) + bait 3389 → Attacks shows **both** ports under spray
- [ ] Under RDP NLA spray, NETWORK must not be the single catch-all bucket
