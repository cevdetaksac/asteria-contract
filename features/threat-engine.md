# Local threat engine — single contract

> **SoT** **≥ 1.4.65**. Cloud IoC bundle is [`threat-intel.md`](./threat-intel.md) (separate).
> Client **≥ 4.9.104**: real-port EventLog fails honor `protection.block_rules` even when honeypot is off.

EventLog / score / `AR-BLOCK` auto-response. Bare RDP success is not
score 100 / auto-block. Whitelist never blocked. Alert hygiene: VSS list is not
ransomware; canary self-touch suppressed.

## Real-port auth (honeypot optional)

1. Windows Security `4625` / TerminalServices / MSSQL fail events feed
   `ThreatEngine` with `target_service` (RDP NLA LogonType 3 → **RDP**, not
   Network when Negotiate/User32 or recent Event 1149).
2. Enabled `protection.block_rules` (or local defaults) count **per service**
   and block at threshold — independent of bait honeypot listen state.
3. Matching fails also POST `/api/attack` (password placeholder
   `<failed_logon>`) so dashboard Attacks / notification counters work without
   honeypot. Bait captures remain a separate channel
   ([`agent/attacks-and-services.md`](../agent/attacks-and-services.md)).
