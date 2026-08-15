# Fleet & version matrix

> Hangi client sürümü hangi contract özelliğini karşılar.  
> Contract VERSION: root `VERSION`.

| Özellik | Contract dosyası | Min client |
|---------|------------------|------------|
| Bearer-only agent API | `api/01-auth.md` | 4.4.33+ / fleet default |
| Control WS commands | `api/03-control-websocket.md` | 4.5.x |
| Self-update GUI banner | `features/self-update.md` | 4.5.39 |
| Remote `inputs[]` piggyback | `features/remote-desktop.md` | **4.5.55** |
| Remote Desktop v2: WS/JPEG healthy path + adaptive telemetry + latest-frame coalescing | `features/remote-desktop.md` | **4.9.0** |
| Remote input protocol 2 + mobile gestures + ACK + persistent session helper | `features/remote-desktop.md` | **4.9.0** |
| Remote Desktop optional WebRTC signaling (protocol=1, non-trickle, H264, data-channel input, cloud-supplied `ice_servers` in offer) | `features/remote-desktop.md` | **4.9.0** |
| Remote Desktop WebRTC smoothness | `features/remote-desktop.md` | client **≥ 4.9.20** |
| WebRTC strict JPEG suppression, 30–60 fps media pacing, optional DXGI capture and effective media telemetry | `features/remote-desktop.md` | **4.9.1** |
| P1 observe schemas: heartbeat_proof, access_integrity, etw_shadow correlation, password_burst, device_identity, canary_coverage, deception_health, network_restore dry_run/rollback | `api/08-architecture.md`, `api/01-auth.md`, `api/03-control-websocket.md` | **4.9.1** (flags default off) |
| OOB-501 offline urgent queue: `POST /api/alerts/urgent/batch` + soft idempotency on urgent | `api/10-offline-urgent-queue.md` | cloud live ≥ **1.4.7**; client flag `security.offline_urgent_queue` default **off** |
| Dashboard Server Management (users/processes/services + `list_services`) | `features/server-management.md` | target ≥ **4.9.4** (cloud UI live ≥ **1.4.8**) |
| On-demand process inspect (`inspect_process` modal) | `features/process-inspect.md` | client **≥ 4.9.93**; cloud whitelist + **no confirm** **1.4.59** |
| Console follow vs lock (`topology=follow` / `winlogon`) | `features/remote-desktop.md` | client **≥ 4.9.95**; pixels + lock/logoff **≥ 4.9.100**; smoothness **≥ 4.9.101**; cloud **1.4.71** |
| Dashboard Users — disabled inventory + enable/disable (`include_disabled:true`, C-USER) | `features/server-management.md` | cloud ≥ **1.4.24**; client list/mutate already ≥ **4.9.4** |
| Account unlink from client Settings (`POST /api/agent/unlink-account`) | `api/02-account.md` | cloud live ≥ **1.4.27**; client **≥ 4.9.26** |
| Account claim gate + unlink email OTP (P0d) | `api/02-account.md` | contract **1.4.48**; client **≥ 4.9.75**; cloud **CL-UNLINK-MAIL** |
| Hardware-bound `machine_id` (MAC+MachineGuid fingerprint; clone split) | `api/01-auth.md` | contract **1.4.26**; client **≥ 4.9.28** |
| In-place token rotate (`POST /api/agent/rotate-token` old→new, same `client_id`) | `api/01-auth.md` | cloud live ≥ **1.4.29**; client **≥ 4.9.31** |
| Brand → Asteria / API base `https://asteria.run` (legacy failover) | `cloud/PRODUCT_BRANDING.md`, `agent/rebrand-asteria.md` | cloud live ≥ **1.4.30**; client **≥ 4.9.32** |
| Firewall brand `HP-*` → `AR-BLOCK-*` / `AR-INTEL-*` (clear+rewrite) | `features/firewall.md`, `api/06-firewall-blocks.md` | cloud live ≥ **1.4.31**; client **≥ 4.9.33** |
| Signing/heartbeat contexts → `asteria-chp-v1` / `asteria-heartbeat-v1` | `agent/rebrand-asteria.md`, `api/03-control-websocket.md`, `api/01-auth.md` | cloud live ≥ **1.4.32**; client **≥ 4.9.35** |
| Anti-brick: account-gated critical auto-actions; silent-hours default OFF; dashboard auto-link; undo mail | `features/account-safety.md`, `api/02-account.md` | cloud ≥ **1.4.34**; client **≥ 4.9.36**; undo_mail_path + rolled_back ≥ **4.9.46** / contract **1.4.38** |
| Cloud / dashboard tick list | `cloud/CLOUD_CHECKLIST.md` | cloud ≥ **1.4.67** (ticked) |
| Client / agent lab tick list | `features/CLIENT_CHECKLIST.md` | client **≥ 4.9.101** RD smoothness · **≥ 4.9.100** RD pixels · **≥ 4.9.95** topo · **≥ 4.9.96** intel/installer |
| Dashboard deep-links catalog (`?token=` browser URLs / Settings shortcuts) | `cloud/dashboard-deep-links.md` | cloud ≥ **1.4.35** (docs; no new client floor) |
| Cloud surface inventory (agent HTTP/WS/commands — client gap-scan) | `cloud/CLOUD_SURFACE.md` | cloud ≥ **1.4.36** (docs; no new client floor) |
| Fleet canary `fleet_rollout` + auto-flag clear | `cloud/FLEET_CANARY.md`, `cloud/PROMOTION_GATES.md` | cloud ≥ **1.4.37**; client honor gates ≥ **4.9.37** |
| Envelope v2 schema (observe-only; no production emit) | `api/12-command-envelope-v2.md` | cloud ≥ **1.4.37**; client observe optional |
| Remote Desktop (topology, follow, Winlogon, CAD, WebRTC, P0, PIX, smoothness) | `features/remote-desktop.md` | client **≥ 4.9.95** topo · **≥ 4.9.100** lock/logon pixels · **≥ 4.9.101** C-RD-SMOOTH (wire ≥4.9.0; S0 helper ≥4.9.84 must not regress) |
| Firewall Management page + `list_firewall` | `features/firewall.md` | client **≥ 4.9.40**; cloud ≥ **1.4.40** |
| Windows Firewall MMC parity | `features/firewall.md` | client **≥ 4.9.41**; cloud ≥ **1.4.41** |
| Network Adapter Admin + golden watchdog | `features/network-adapter.md` | client **≥ 4.9.42**; cloud ≥ **1.4.42** |
| Windows Tools Repair (remote OS repair toolkit) | `features/tools-repair.md` | client **≥ 4.9.79**; cloud ≥ **1.4.49** |
| Legacy dual-brand sunset target **2026-10-01** | `cloud/PRODUCT_BRANDING.md` | ops + client ≥**4.9.35** |
| Uninstall PIN lifecycle (`uninstall_*` + `windows_user`) | `api/07-lifecycle-sessions.md` | cloud ≥ **1.4.9**; client with uninstall PIN gate |
| Bare `successful_logon` score ≤70 + no auto-block (cloud reject shapes) | `features/threat-engine.md` | cloud ≥ **1.4.11**; target client **≥4.9.7** |
| Whitelist never stays blocked (reject + `unblock_ip` + pending-unblocks) | `api/06-firewall-blocks.md`, `features/threat-engine.md` | cloud ≥ **1.4.11**; client must ACK `block-removed` |
| Realtime presence (`presence`/`goodbye` + `POST /api/presence`) | `api/11-presence-realtime.md` | cloud live ≥ **1.4.27**; target client **≥ 4.9.8** |
| System Recovery (policy/service/firewall allowlist snapshot + drift + confirm restore) | `features/system-recovery.md` | cloud ≥ **1.4.13**; target client **≥ 4.9.12** |
| Network Guard dashboard panel (live IP/adapters + golden baseline + diff + restore) + `auto_restore_network` | `features/network-guard.md` | cloud ≥ **1.4.14**; target client **≥ 4.9.12** |
| Network Guard maintenance mode (pause → change VPN/IP → snapshot → resume) | `features/network-guard.md` | cloud ≥ **1.4.15**; target client **≥ 4.9.12** |
| VSS delete intent (kill + quarantine without shadow-count wait) | `features/ransomware.md` | cloud ≥ **1.4.16**; target client **≥ 4.9.14** |
| Network Guard soft surface inform (additive adapter/DHCP notify; Accept/Disable; no panic while online) | `features/network-guard.md` | cloud ≥ **1.4.17**; target client **≥ 4.9.15** |
| Defense Policy (tiered + observe default + auto-promote) | `features/defense-policy.md`, `ROADMAP_TIERED_DEFENSE.md` | cloud ≥ **1.4.19**; client **≥ 4.9.17** (matrix ≥ **4.9.16**) |
| One GUI/session + motor watchdog | `api/08-architecture.md` | 4.5.58–59 |
| Threat-intel GET/304/ACK | `features/threat-intel.md` | **4.5.61** |
| Threat-intel `HP-INTEL-*` apply + orphan/policy reconcile + ACK skipped/removed | `features/threat-intel.md`, `api/06-firewall-blocks.md` | **4.9.7** |
| Bare successful_logon no auto HP-BLOCK; silent-hours alert-only FW; whitelist enforce unblock | `features/threat-engine.md`, `features/gui.md` | **4.9.7** |
| Canary sort-bait + quarantine IPC | `features/ransomware.md` | 4.5.62–64 |
| Canary UX (no local scare, OneDrive skip) | `features/ransomware.md` | **4.5.65** |
| `protection.block_rules` apply + WS intel push | `agent/register-protection.md`, `api/09` | **4.5.66** |
| Enriched canary alert + quarantine snapshot | `features/ransomware.md` | 4.5.67 |
| Single enriched canary urgent (thin-race fix) | `features/ransomware.md` | **4.5.68** |
| Guardian service + cross-watchdog + tamper wire | `features/persistence.md` | **4.6.0** |
| `create_user` + `remote_logon` (autologon break-glass) | `features/disaster-recovery.md` | **4.6.0** |
| Network Guard baseline + offline şüpheli tespiti | `features/network-guard.md` | **4.7.0** |
| Network Guard locale decode hotfix | `features/network-guard.md` | **4.7.1** |
| Network Guard false-positive fix + alert-only default | `features/network-guard.md` | **4.7.2** |
| Hard alert-only + onaylı exact-process suspend/resume | `features/network-guard.md` | **4.7.3** |
| GUI Security Layers + cloud config push + sayaç invariant | `features/gui.md` | **4.7.3** |
| Daemon STATUS recursion / GUI-Guardian IPC health hotfix | `api/08-architecture.md` | **4.7.4** |
| Update lock → replacement motor-ready tamper handoff | `api/08-architecture.md` | **4.7.5** |
| Daily client/threat/lifecycle logs + 7-day retention | `agent/log-retention.md` | **4.7.6** |
| Protection strip + Settings tab + STATUS network_guard/rs alanları | `features/gui.md`, `api/08` | **4.8.0** |
| Detay popup veri-kaynağı invariantı (chip=popup, daemon STATUS) | `features/gui.md` | **4.8.1** |
| Settings webhook cloud→local köprüsü + config surface dokümante | `features/threat-engine.md` | **4.8.2** |
| Dashboard PIN set/reset (`set_gui_pin`/`clear_gui_pin`) + IP hızlı aksiyon butonları | `api/03-control-websocket.md`, `features/gui.md` | **4.8.3** |
| Whitelist cloud SoT (merge-only persist + tablo cloud okur) | `features/gui.md` | **4.8.4** |
| block-removed ACK ips(+ids); dashboard "Kaldırılıyor" takılma fix | `api/06-firewall-blocks.md` | **4.8.5** |

**Önerilen production floor:** **4.9.0** (RD v2 + recovery + NG containment + GUI control center + whitelist SoT + unblock ACK).

Avoid in production: **4.7.0–4.7.2** (NG FP); prefer **≥4.7.3** containment, **≥4.8.1** popup SoT, **≥4.8.5** unblock ACK, **≥4.9.0** RD v2.

Cloud publish: contract clone → `git pull` → `scripts/publish_contract.sh` (HTTPS mirror).
