# Contract INDEX

> Oku: [`VERSION`](VERSION) → bu dosya → satırdaki MD.  
> **VERSION 1.4.61** · Repo: https://github.com/cevdetaksac/asteria-contract
> Fleet: [`FLEET.md`](FLEET.md) · Production floor: **client ≥ 4.9.0**  
> **API base:** `https://asteria.run` (legacy alias details: [`agent/rebrand-asteria.md`](agent/rebrand-asteria.md))

### Recent contract highlights

| Contract | Konu | Client |
|----------|------|--------|
| **1.4.61** | Fold remaining products into `features/*` | [`features/README.md`](features/README.md) |
| **1.4.58** | Console-first RD: default Logon; follow Default after logon | **≥4.9.93**; cloud default winlogon |
| **1.4.57** | On-demand `inspect_process` + rundll32 LOLBIN false-positive fix | **≥4.9.93** |
| **1.4.56** | Logon lab: live meta after settle (not start snapshot); `phase=degraded` | **≥4.9.91**; cloud live meta |
| **1.4.55** | Logon chrome pixels — solid blue ≠ Winlogon success | **≥4.9.87** |
| **1.4.54** | CAD meta lab close — sas_gen int + ui≠unknown + effect (`4.9.86`) | ≥**4.9.86** (meta accepted) |
| **1.4.53** | CAD honesty lab note — `SAS_NO_EFFECT` OK; real SAS UI still open | honesty **4.9.85**; effect → **≥4.9.86** |
| **1.4.52** | CAD / Winlogon input honesty — no false SendSAS success | **≥4.9.85**; cloud CAD poll shipped |
| **1.4.51** | C-RD-S0 lab close — Session-0 Logon pixels on **4.9.84** | ≥**4.9.84** (accepted) |
| **1.4.50** | Winlogon Session-0 capture P0 (Logon UI black / jpeg=0B helper) | **≥4.9.84**; cloud omits SID on Logon Start |
| **1.4.49** | Windows Tools Repair — remote OS repair toolkit | ≥**4.9.79**; cloud whitelist + dashboard |
| **1.4.48** | Anti-brick cloud P0 + claim gate + unlink email magic-link (P0d) | cloud shipped; client ≥**4.9.76** |
| **1.4.47** | RD console viewer software cursor (C-RD-VIEW) — Proxmox-like without noVNC | dashboard; client ≥**4.9.49** |
| **1.4.46** | Self-update progress ticks | ≥**4.9.60** |
| **1.4.43** | Console RD Winlogon physical-parity (C-RD-CON) | ≥**4.9.49** |
| **1.4.38** | Anti-brick undo_mail_path wire + C-BRICK-1.3/6 client; cloud backlog | ≥**4.9.46** |
| **1.4.37** | Fleet canary · envelope v2 schema · RD P0 · brand sunset · promote gates | canary/RD ≥**4.9.37**; schema docs all |
| **1.4.36** | Cloud surface inventory — agent HTTP/WS/commands gap-scan | — (docs) |
| **1.4.35** | Dashboard deep-links (`?token=`) catalog for Settings shortcuts | — (docs) |
| **1.4.34** | Anti-brick: account-gated critical auto-actions + dashboard auto-link + undo mail | ≥**4.9.36** |
| **1.4.33** | Contract hygiene: client entrypoints, remove legacy archive dumps | — (docs) |
| **1.4.32** | Signing/heartbeat → **`asteria-chp-v1` / `asteria-heartbeat-v1`**; brand SoT scrub | ≥**4.9.35** |
| **1.4.31** | Firewall brand `HP-*` → **`AR-BLOCK` / `AR-INTEL`** | ≥**4.9.33** |
| **1.4.30** | Brand → **Asteria** / `asteria.run` (API base cutover) | ≥**4.9.32** |
| **1.4.29** | In-place token rotate (`old→new`, same `client_id`) | ≥**4.9.31** |
| **1.4.27** | Unlink API live + realtime presence wire | unlink ≥**4.9.26**; presence ≥**4.9.8** |
| **1.4.26** | Hardware-bound `machine_id` (clone split) | ≥**4.9.28** |
| **1.4.25** | Account unlink Spec (Settings) | ≥**4.9.26** |
| **1.4.24** | Server Users C-USER | cloud; list ≥**4.9.4** |
| **1.4.23** | Winlogon sibling pre_logon + C-WL | ≥**4.9.26** |
| **1.4.20** | WebRTC smoothness | ≥**4.9.20** |
| **1.4.19** | Defense Policy observe→balanced | ≥**4.9.17** |
| Schema | ZT envelope v2 observe-only — [`api/12-command-envelope-v2.md`](api/12-command-envelope-v2.md) | observe |
| Decision | Branding + sunset **2026-10-01** — [`cloud/PRODUCT_BRANDING.md`](cloud/PRODUCT_BRANDING.md) | ≥**4.9.35** |

---

## Shared delivery plans

| Dosya | Konu | Statü |
|-------|------|-------|
| [ROADMAP_TIERED_DEFENSE.md](ROADMAP_TIERED_DEFENSE.md) | Kademeli savunma, onboarding, anti-bait | Planning (living) |
| [SECURITY_RESILIENCE_VNEXT.md](SECURITY_RESILIENCE_VNEXT.md) | Ortak güvenlik paketleri; §7A observe şemaları api/agent’ta | Plan + promoted observe |

---

## features/ (one file per product)

| Dosya | Konu | Min client |
|-------|------|------------|
| [features/README.md](features/README.md) | One MD per product | — |
| [features/remote-desktop.md](features/remote-desktop.md) | RD | **≥ 4.9.95** |
| [features/self-update.md](features/self-update.md) | Self-update trust/asset/progress | **≥ 4.9.96** |
| [features/process-inspect.md](features/process-inspect.md) | `inspect_process` | **≥ 4.9.93** |
| [features/server-management.md](features/server-management.md) | Users / processes / services | **≥ 4.9.4** |
| [features/threat-intel.md](features/threat-intel.md) | Intel bundle ACK | **≥ 4.9.96** |
| [features/firewall.md](features/firewall.md) | AR-BLOCK / inventory / MMC | **≥ 4.9.41** |
| [features/defense-policy.md](features/defense-policy.md) | Observe / balanced / isolate | **≥ 4.9.17** |
| [features/network-guard.md](features/network-guard.md) | Alert-only + restore confirm | **≥ 4.7.3** |
| [features/account-safety.md](features/account-safety.md) | Link / unlink / anti-brick | **≥ 4.9.46** |
| [features/service-port.md](features/service-port.md) | Relocate listen port | **≥ 4.9.45** |

Pointers (do not extend): `api/05-remote-desktop.md` (envelopes), `agent/remote-input.md` (input appendix), `cloud/remote-console-parity.md` and other `*remote*` / `*winlogon*` stubs.

---

| Dosya | Konu | Min client |
|-------|------|------------|
| [agent/CLIENT.md](agent/CLIENT.md) | İndeks / okuma sırası | ≥ **4.5.68** |
| [agent/polling.md](agent/polling.md) | Cadence tablosu | — |
| [agent/register-protection.md](agent/register-protection.md) | `protection.block_rules` | ≥ **4.5.66** |
| [agent/ransomware-shield.md](agent/ransomware-shield.md) | Canary UX, quarantine, unlock | ≥ **4.5.65** |
| [agent/persistence-and-tamper.md](agent/persistence-and-tamper.md) | Guardian, watchdog, tamper | ≥ **4.6.0** |
| [agent/disaster-recovery.md](agent/disaster-recovery.md) | `create_user`, `remote_logon` | ≥ **4.6.0** |
| [agent/network-guard.md](agent/network-guard.md) | Offline alarm, golden network, soft inform, contain | ≥ **4.7.3** (panel ≥**4.9.12**; soft ≥**4.9.15**) |
| [agent/system-recovery.md](agent/system-recovery.md) | Surface snapshot / drift / restore | target ≥ **4.9.12** |
| [agent/gui-control-center.md](agent/gui-control-center.md) | GUI katmanlar, şerit, Ayarlar, popup SoT | ≥ **4.7.3** (şerit ≥**4.8.0**) |
| [agent/log-retention.md](agent/log-retention.md) | Yerel log 7 gün | ≥ **4.7.6** |
| [agent/attacks-and-services.md](agent/attacks-and-services.md) | Attack, bait tunnels, ports | — |
| [agent/service-port-relocate.md](agent/service-port-relocate.md) | One-click real-service → safe port (cloud base) | ≥4.9.44 |
| [agent/service-port-relocate-client.md](agent/service-port-relocate-client.md) | Bidirectional sync + GUI relocate-report | ≥**4.9.45** |
| [agent/self-update-progress.md](agent/self-update-progress.md) | self_update download % / phase ticks | ≥**4.9.60** |
| [agent/threat-engine.md](agent/threat-engine.md) | Urgent/batch/health/config; whitelist enforce | ≥ **4.9.7** |
| [agent/defense-policy-client.md](agent/defense-policy-client.md) | Matrix apply, observe default, CTA | ≥ **4.9.17** |
| [agent/remote-input.md](agent/remote-input.md) | Input envelope appendix → [`features/remote-desktop.md`](features/remote-desktop.md) | ≥ **4.9.0** |
| [agent/remote-desktop-p0.md](agent/remote-desktop-p0.md) | Pointer → feature RD | ≥ **4.9.37** |
| [agent/server-management.md](agent/server-management.md) | Users / processes / services | target ≥ **4.9.4** |
| [agent/process-inspect.md](agent/process-inspect.md) | On-demand process evidence (`inspect_process`) | ≥ **4.9.93** |
| [agent/rebrand-asteria.md](agent/rebrand-asteria.md) | Host, UI, signing/heartbeat Asteria cutover | ≥ **4.9.35** |
| [agent/firewall-brand-migrate.md](agent/firewall-brand-migrate.md) | `HP-*` → `AR-BLOCK` / `AR-INTEL` | ≥ **4.9.33** |
| [agent/anti-brick-critical-actions.md](agent/anti-brick-critical-actions.md) | Account-gated critical auto + anti-brick (C-BRICK) | ≥ **4.9.46** (1.3/6); floor **4.9.36** |
| [agent/firewall-management.md](agent/firewall-management.md) | Asteria FW inventory + dashboard page (1.4.40) | ≥ **4.9.40** |
| [agent/firewall-windows-parity.md](agent/firewall-windows-parity.md) | Windows FW MMC parity (all rules + profiles + mutate) | ≥ **4.9.41** |
| [agent/network-adapter-admin.md](agent/network-adapter-admin.md) | Adapter enable/IP/DNS + golden watchdog rollback | ≥ **4.9.42** |
| [agent/tools-repair.md](agent/tools-repair.md) | Remote Windows repair toolkit (share/print/SFC/…) | ≥ **4.9.79** |

---

## api/

| Dosya | Konu | Min client |
|-------|------|------------|
| [api/01-auth.md](api/01-auth.md) | Bearer, register, heartbeat, `machine_id`, **rotate-token** | 4.4.33+ / hw-id ≥**4.9.28** / rotate ≥**4.9.31** |
| [api/02-account.md](api/02-account.md) | Account link / unlink / multi-server / dashboard auto-link | unlink ≥**4.9.26**; auto-link ≥**1.4.34** |
| [api/03-control-websocket.md](api/03-control-websocket.md) | Control WS + komutlar + HMAC (`asteria-chp-v1`) | 4.5.x / signing ≥**4.9.35** |
| [api/04-self-update.md](api/04-self-update.md) | Self-update ACK | 4.5.39+ |
| [api/05-remote-desktop.md](api/05-remote-desktop.md) | RD WS envelope appendix → [`features/remote-desktop.md`](features/remote-desktop.md) | **≥4.9.0** |
| [api/06-firewall-blocks.md](api/06-firewall-blocks.md) | AR-BLOCK / sync / clear (+ HP wipe) | 4.5.40+ / brand ≥**4.9.33** |
| [api/07-lifecycle-sessions.md](api/07-lifecycle-sessions.md) | Lifecycle / sessions / processes | — |
| [api/08-architecture.md](api/08-architecture.md) | Daemon vs GUI IPC | ≥ **4.5.66** |
| [api/09-threat-intel.md](api/09-threat-intel.md) | Intel bundle ETag/ACK/WS | ≥ **4.5.61** / apply ≥**4.9.7** |
| [api/10-offline-urgent-queue.md](api/10-offline-urgent-queue.md) | OOB-501 batch + idempotency + promote criteria | cloud live; client flag **off**; canary **1.4.37** |
| [api/11-presence-realtime.md](api/11-presence-realtime.md) | Sleep/suspend/shutdown presence | cloud live ≥**1.4.27**; client ≥**4.9.8** |
| [api/12-command-envelope-v2.md](api/12-command-envelope-v2.md) | ZT-601 envelope schema (observe-only; no emit) | observe |

---

## cloud/

| Dosya | Konu | Statü |
|-------|------|-------|
| [cloud/overview.md](cloud/overview.md) | Mimari özet (kısa) | Normative |
| [cloud/CLOUD_SURFACE.md](cloud/CLOUD_SURFACE.md) | Agent HTTP/WS/commands inventory — client gap-scan | Normative (1.4.36) |
| [cloud/DEFENSE_POLICY.md](cloud/DEFENSE_POLICY.md) | Tiered policy + auto-promote (C-P0…) | Normative (1.4.19) |
| [cloud/SERVER_USER_MANAGEMENT.md](cloud/SERVER_USER_MANAGEMENT.md) | Users C-USER-1…7 | Normative (1.4.24) |
| [cloud/REMOTE_DESKTOP_WINLOGON.md](cloud/REMOTE_DESKTOP_WINLOGON.md) | Winlogon / pre_logon C-WL | Normative (1.4.23) |
| [cloud/REMOTE_DESKTOP_SMOOTHNESS.md](cloud/REMOTE_DESKTOP_SMOOTHNESS.md) | WebRTC smoothness C-RD | Normative (1.4.20) |
| [cloud/threat-intel-ingest.md](cloud/threat-intel-ingest.md) | Harici feed → bundle | Normative |
| [cloud/PRODUCT_BRANDING.md](cloud/PRODUCT_BRANDING.md) | Wire identity / no big-bang rename | Decision |
| [cloud/dashboard-deep-links.md](cloud/dashboard-deep-links.md) | Browser `?token=` dashboard URLs / Settings shortcuts | Normative (1.4.35) |
| [cloud/PROMOTION_GATES.md](cloud/PROMOTION_GATES.md) | Observe→enforce criteria (single SoT) | Normative (1.4.37) |
| [cloud/FLEET_CANARY.md](cloud/FLEET_CANARY.md) | `fleet_rollout{}` canary gates | Normative (1.4.37) |
| [cloud/CLOUD_BACKLOG.md](cloud/CLOUD_BACKLOG.md) | Cloud-owned P0/P1 remaining work | Status (1.4.38) |
| [cloud/ZERO_TRUST_STATUS.md](cloud/ZERO_TRUST_STATUS.md) | ZT do/don’t | Status (1.4.37) |
| [cloud/command-envelope-v2-design.md](cloud/command-envelope-v2-design.md) | ZT-601 design archive (schema → api/12) | Archive |
| [cloud/operator-keyset-design.md](cloud/operator-keyset-design.md) | ZT-602/603 keys | Design-only |

---

## Opsiyonel lab (zorunlu değil)

- Cadence: `agent/polling.md` canlı host doğrulama  
- Live host: 3 fail → `AR-BLOCK-*` smoke (`agent/register-protection.md`)
