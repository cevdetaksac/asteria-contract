# Threat intel — single contract

> **SoT** **≥ 1.4.61** · Client poll **≥ 4.5.61** · apply **≥ 4.9.7** · `AR-INTEL-*` **≥ 4.9.33**
> · `firewall_current` + 304-no-ACK **≥ 4.9.96**
> Pointers: [`../api/09-threat-intel.md`](../api/09-threat-intel.md),
> [`../cloud/threat-intel-ingest.md`](../cloud/threat-intel-ingest.md)
> Local EventLog engine is **not** this file (`agent/threat-engine.md`).

Client **does not** fetch Abuse.ch / CISA / ThreatFox. Cloud ingest only.

`GET /api/agent/threat-intel` + `If-None-Match`.

| Code | Agent |
|------|--------|
| **200** | Save bundle, apply layers, **ACK** |
| **304** | Keep cache, reconcile expire/orphan, **no ACK** |
| 404/5xx | Soft-fail |

ACK `stats`: `firewall_added` / `skipped` / `removed` / **`firewall_current`** (standing `AR-INTEL-*` count) / ransomware / process_watch / banners / errors.

WS `threat_intel_updated` → GET immediately.

Apply: `AR-INTEL-<id>` in+out (not `AR-BLOCK` / AutoResponse 24h). `auto_block_firewall=false` purges intel rules.
