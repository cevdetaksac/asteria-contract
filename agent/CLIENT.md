# Windows Client — sözleşme indeksi

> **Repo:** [cevdetaksac/asteria-contract](https://github.com/cevdetaksac/asteria-contract)  
> **VERSION:** root [`VERSION`](../VERSION) · giriş: [`INDEX.md`](../INDEX.md) · fleet: [`FLEET.md`](../FLEET.md)  
> **API:** `https://asteria.run`  
> **Brand / signing SoT:** [`rebrand-asteria.md`](./rebrand-asteria.md) — contract **≥ 1.4.32**, client **≥ 4.9.35**  
> **Production floor:** client ≥ **4.9.0** (feature floors in `FLEET.md`)

Bu dosya **özet + link**. Şema/detay için ilgili MD’ye git; buraya kopyalama.

---

## Okuma sırası (yeni agent / sprint)

0. [`rebrand-asteria.md`](./rebrand-asteria.md) — host, UI, **`asteria-chp-v1` / `asteria-heartbeat-v1`**, path trust (**1.4.32+**, ≥ **4.9.35**)  
0b. [`firewall-brand-migrate.md`](./firewall-brand-migrate.md) — `AR-BLOCK` / `AR-INTEL` (**1.4.31**, ≥ **4.9.33**)  
0c. [`firewall-management.md`](./firewall-management.md) — Asteria inventory + dashboard page (**1.4.40**, ≥ **4.9.40**)
0c2. [`firewall-windows-parity.md`](./firewall-windows-parity.md) — Windows FW MMC parity (**1.4.41**, ≥ **4.9.41**)
0c3. [`network-adapter-admin.md`](./network-adapter-admin.md) — adapter IP/DNS + golden watchdog (**1.4.42**, ≥ **4.9.42**)
0d. [`anti-brick-critical-actions.md`](./anti-brick-critical-actions.md) — account-gated critical auto / anti-brick (**1.4.34**, ≥ **4.9.36**)  
0e. [`../cloud/dashboard-deep-links.md`](../cloud/dashboard-deep-links.md) — Settings / tray browser shortcuts (`?token=`) (**1.4.35**)  
0f. [`../cloud/CLOUD_SURFACE.md`](../cloud/CLOUD_SURFACE.md) — **cloud yetenekleri / HTTP+WS+komut envanteri** (gap-scan) (**1.4.36**)  
0g. [`../cloud/FLEET_CANARY.md`](../cloud/FLEET_CANARY.md) + [`../cloud/PROMOTION_GATES.md`](../cloud/PROMOTION_GATES.md) — automation canary / observe→enforce (**1.4.37**, ≥ **4.9.37**)  
0h. [`remote-desktop-p0.md`](./remote-desktop-p0.md) — **P0** Winlogon black + ICE honesty (**1.4.37**, ≥ **4.9.37**)  
0i. [`../api/12-command-envelope-v2.md`](../api/12-command-envelope-v2.md) — envelope v2 schema (observe-only; **no emit**) (**1.4.37**)  
1. [`../FLEET.md`](../FLEET.md) — min sürüm matrisi  
2. [`polling.md`](./polling.md) — cadence  
3. [`../api/01-auth.md`](../api/01-auth.md) — register / heartbeat / Bearer / rotate-token  
4. [`register-protection.md`](./register-protection.md) — `protection.block_rules`  
5. [`attacks-and-services.md`](./attacks-and-services.md) — bait + attack POST  
6. [`../api/03-control-websocket.md`](../api/03-control-websocket.md) — WS + **komut kataloğu** + HMAC  
7. [`threat-engine.md`](./threat-engine.md) — v4 alerts/health/config  
8. [`../api/09-threat-intel.md`](../api/09-threat-intel.md) — cloud IoC bundle  
9. [`ransomware-shield.md`](./ransomware-shield.md) — canary / quarantine / unlock  
10. [`remote-input.md`](./remote-input.md) + [`../api/05-remote-desktop.md`](../api/05-remote-desktop.md)  
11. [`persistence-and-tamper.md`](./persistence-and-tamper.md) — survival + tamper (≥4.6.0)  
12. [`disaster-recovery.md`](./disaster-recovery.md) — `create_user` / `remote_logon` (≥4.6.0)  
13. [`network-guard.md`](./network-guard.md) — golden ağ + soft inform + contain (≥4.7.3 / panel ≥4.9.12 / soft ≥4.9.15)  
13b. [`../cloud/DEFENSE_POLICY.md`](../cloud/DEFENSE_POLICY.md) + [`defense-policy-client.md`](./defense-policy-client.md) — tiered policy (**1.4.19**, ≥4.9.17)  
14. [`system-recovery.md`](./system-recovery.md) — surface snapshot / drift / restore (target ≥4.9.12)  
15. [`gui-control-center.md`](./gui-control-center.md) — GUI güvenlik katmanları (≥4.7.3)  
15b. [`gui-webview-bridge.md`](./gui-webview-bridge.md) — GUI process split draft (design)  
16. [`log-retention.md`](./log-retention.md) — yerel log 7 gün (≥4.7.6)  
17. [`server-management.md`](./server-management.md) — users / processes / services (target ≥4.9.4)  
18. [`../api/11-presence-realtime.md`](../api/11-presence-realtime.md) — sleep/shutdown presence (≥4.9.8; cloud ≥1.4.27)  
19. [`../api/06-firewall-blocks.md`](../api/06-firewall-blocks.md) · [`04-self-update.md`](../api/04-self-update.md) · [`07-lifecycle-sessions.md`](../api/07-lifecycle-sessions.md) · [`08-architecture.md`](../api/08-architecture.md) · [`02-account.md`](../api/02-account.md)

---

## Auth (tek satır)

Bearer only. Token ProgramData. Query `?token=` gönderme.

- **İlk kurulum:** `POST /api/register`  
- **Token yeniden üretim:** `POST /api/agent/rotate-token` (`old_token`→`new_token`, aynı `client_id`) — **≥ 4.9.31** / **1.4.29**. Bare register ile yeni satır açma.
- **Signing:** command `asteria-chp-v1`, heartbeat `asteria-heartbeat-v1` — **≥ 4.9.35**.

---

## Control WS (tek satır)

`wss://asteria.run/ws/agent/control` → `command` push; result HTTP; `threat_intel_updated` → anında intel GET.

---

## IR notu

`contain_user` cloud whitelist’te **yok** → `logoff_user` + `reset_password` (+ `disable_account`).  
Tam liste: [`../api/03-control-websocket.md`](../api/03-control-websocket.md).

**Anti-brick (≥4.9.36):** `account_linked` değilse yerel kritik auto (`disable_account` / silent-hours disable) **yasak** — fail-closed skip. Detay: [`anti-brick-critical-actions.md`](./anti-brick-critical-actions.md).

---

## Ransomware (tek satır)

Canary Hidden+System; yerel scare yok; unlock = GUI / `RS_UNLOCK` / `unlock_ransomware_quarantine`. Detay: [`ransomware-shield.md`](./ransomware-shield.md).

---

## Mimari (tek satır)

SYSTEM daemon = motor (WS, FW, update, intel, ransomware). GUI = tray/UI + IPC (`RS_*`). Detay: `api/08-architecture.md`.

---

## Survival + kurtarma (tek satır, ≥4.6.0)

Motor+Guardian çapraz watchdog; durdurma yalnız update-lock / imzalı PIN; başka çıkış → tamper + diriliş. Felaket: `create_user` + `remote_logon`. Detay: [`persistence-and-tamper.md`](./persistence-and-tamper.md), [`disaster-recovery.md`](./disaster-recovery.md).

---

## Network Guard + GUI (tek satır, ≥4.7.3)

Şüpheli yoğun aktivite **yalnız alarm**; süreç dondurma onaylı `suspend_process`. Katman toggle’ları GUI → `POST /api/threats/config`. Detay: [`network-guard.md`](./network-guard.md), [`gui-control-center.md`](./gui-control-center.md).

---

## Remote Desktop v2 (tek satır, ≥4.9.0)

Sağlıklı yol **WS + JPEG**; input protocol 2; opsiyonel WebRTC → JPEG fallback. Detay: [`../api/05-remote-desktop.md`](../api/05-remote-desktop.md), [`remote-input.md`](./remote-input.md).
