# Contract INDEX

> [`VERSION`](VERSION) (**1.4.65**) → [`features/README.md`](features/README.md) → that product MD.  
> Repo: https://github.com/cevdetaksac/asteria-contract · Fleet: [`FLEET.md`](FLEET.md)  
> Production floor: **client ≥ 4.9.0** · API: `https://asteria.run`  
> **Cloud work:** [`cloud/CLOUD_CHECKLIST.md`](cloud/CLOUD_CHECKLIST.md)

Conflict: **`features/*` wins.** `agent/` `api/` `cloud/` are shared wire or appendices.

---

## features/

| Dosya | Min client |
|-------|------------|
| [features/README.md](features/README.md) | — |
| [features/remote-desktop.md](features/remote-desktop.md) | **≥ 4.9.95** |
| [features/self-update.md](features/self-update.md) | **≥ 4.9.96** |
| [features/process-inspect.md](features/process-inspect.md) | **≥ 4.9.93** |
| [features/server-management.md](features/server-management.md) | **≥ 4.9.4** |
| [features/threat-intel.md](features/threat-intel.md) | **≥ 4.9.96** |
| [features/firewall.md](features/firewall.md) | **≥ 4.9.41** |
| [features/defense-policy.md](features/defense-policy.md) | **≥ 4.9.17** |
| [features/network-guard.md](features/network-guard.md) | **≥ 4.7.3** |
| [features/account-safety.md](features/account-safety.md) | **≥ 4.9.46** |
| [features/service-port.md](features/service-port.md) | **≥ 4.9.45** |
| [features/ransomware.md](features/ransomware.md) | **≥ 4.5.65** |
| [features/persistence.md](features/persistence.md) | **≥ 4.6.0** |
| [features/disaster-recovery.md](features/disaster-recovery.md) | **≥ 4.6.0** |
| [features/system-recovery.md](features/system-recovery.md) | **≥ 4.9.12** |
| [features/tools-repair.md](features/tools-repair.md) | **≥ 4.9.79** |
| [features/network-adapter.md](features/network-adapter.md) | **≥ 4.9.42** |
| [features/gui.md](features/gui.md) | **≥ 4.8.0** |
| [features/presence.md](features/presence.md) | **≥ 4.9.8** |
| [features/threat-engine.md](features/threat-engine.md) | **≥ 4.9.7** |

---

## Shared wire (`api/`)

| Dosya | Konu |
|-------|------|
| [api/01-auth.md](api/01-auth.md) | Bearer, register, heartbeat, rotate-token |
| [api/02-account.md](api/02-account.md) | Link / unlink |
| [api/03-control-websocket.md](api/03-control-websocket.md) | Control WS + command catalog + HMAC |
| [api/06-firewall-blocks.md](api/06-firewall-blocks.md) | AR-BLOCK sync |
| [api/07-lifecycle-sessions.md](api/07-lifecycle-sessions.md) | Lifecycle / sessions |
| [api/08-architecture.md](api/08-architecture.md) | Daemon vs GUI IPC |
| [api/10-offline-urgent-queue.md](api/10-offline-urgent-queue.md) | OOB-501 (client flag off) |
| [api/11-presence-realtime.md](api/11-presence-realtime.md) | Sleep / goodbye |
| [api/12-command-envelope-v2.md](api/12-command-envelope-v2.md) | Observe-only; no emit |

`api/04`, `api/05`, `api/09` are stubs → `features/self-update.md`, `features/remote-desktop.md`, `features/threat-intel.md`.

---

## Cloud ops

| Dosya | Konu |
|-------|------|
| [cloud/CLOUD_CHECKLIST.md](cloud/CLOUD_CHECKLIST.md) | **Tick list for dashboard/API** |
| [cloud/CLOUD_SURFACE.md](cloud/CLOUD_SURFACE.md) | HTTP/WS inventory |
| [cloud/dashboard-deep-links.md](cloud/dashboard-deep-links.md) | `?token=` Settings URLs |
| [cloud/FLEET_CANARY.md](cloud/FLEET_CANARY.md) | Canary gates |
| [cloud/PROMOTION_GATES.md](cloud/PROMOTION_GATES.md) | Observe→enforce |
| [cloud/PRODUCT_BRANDING.md](cloud/PRODUCT_BRANDING.md) | Sunset 2026-10-01 |
| [cloud/threat-intel-ingest.md](cloud/threat-intel-ingest.md) | External feeds (cloud-only) |
| [cloud/overview.md](cloud/overview.md) | Architecture sketch |

`cloud/DEFENSE_POLICY.md` and `cloud/SERVER_USER_MANAGEMENT.md` are stubs → `features/defense-policy.md`, `features/server-management.md`.

---

## Agent appendices

Index: [`agent/CLIENT.md`](agent/CLIENT.md) · brand: [`agent/rebrand-asteria.md`](agent/rebrand-asteria.md).  
Product files under `agent/` are stubs. Unique leftovers: `rebrand-asteria.md`, `register-protection.md`, `polling.md`, `log-retention.md`, `attacks-and-services.md`, `gui-webview-bridge.md` (draft).  
Do not add new MUST IDs under `agent/` — extend `features/` instead.

---

## Plans (not product SoT)

[ROADMAP_TIERED_DEFENSE.md](ROADMAP_TIERED_DEFENSE.md) · [SECURITY_RESILIENCE_VNEXT.md](SECURITY_RESILIENCE_VNEXT.md) · [FLEET.md](FLEET.md)
