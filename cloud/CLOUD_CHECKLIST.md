# Cloud / dashboard checklist

> Contract **≥ 1.4.72**. Cloud this file, tick `[x]`, PR note with commit/date.
> Do **not** add new SoT markdown here — implement against `features/*`.
>
> Pin: client **≥ 4.9.103** recommended (PIX DXGI + SMOOTH) · topology **≥ 4.9.95** · inspect **≥ 4.9.93**
> API: `https://asteria.run` · HMAC `asteria-chp-v1`

**Read first:** [`../features/README.md`](../features/README.md)  
**Shared wire:** [`../api/01-auth.md`](../api/01-auth.md) · [`../api/03-control-websocket.md`](../api/03-control-websocket.md)

How to mark: `[x]` = verified in **production** dashboard/API (not just code review).

---

## P0 — Remote Desktop

SoT: [`../features/remote-desktop.md`](../features/remote-desktop.md)

- [x] **C-RD-TOPO-1** Default Connect / “Logon · varsayılan” Start: `{ topology:"follow", stream_id }` only. Do **not** send `prefer`, `pre_logon`, `desktop`, `session_id`, `username`. **1.4.71:** also send `fps:30`, `quality:72`, `max_width:1920` (see SMOOTH-1).
- [x] **C-RD-TOPO-2** Lock / Logon **row**: `topology=winlogon` + `prefer=winlogon` + `pre_logon=true` + `desktop=Winlogon`. Omit SID and username.
- [x] **C-RD-TOPO-3** User shortcut: `session_id` + `username`. Never auto-pick first Active SID.
- [x] **C-RD-TOPO-4** After Enter: **same `stream_id`**, no second Start, no “pick administrator”.
- [x] **C-RD-TOPO-5** Warn agent &lt;4.9.26; 4.9.94 follow-skip is not acceptance.
- [x] **C-RD-TOPO-5 / VIEW-9 (1.4.70/71)** Recommend **≥4.9.103**. Banner copy must not say “≥4.9.45 P0” as the current floor. *(prod 2026-08-17 asteria.run — Derin-Web 4.9.103 banner hidden; i18n `?v=20260817-c172`; no 4.9.45 string)*
- [x] **C-RD-SMOOTH-1 (1.4.71)** Every `remote_stream_start` uses `fps≥30`, `quality≥72`, `max_width:1920`. Remove `fps:12` / `quality:40` / `max_width:1280` from dashboard defaults and samples. *(prod 2026-08-17 — Start `{fps:30,quality:72,max_width:1920}`; FPS select 30/45/60 only; API clamp)*
- [x] **C-RD-SMOOTH-2 (1.4.71)** WebRTC offer includes `ice_servers` with **TURNS on 443** (and STUN). Host UDP through Cloudflare is often dead; TCP 443 alone is not WebRTC media. *(prod 2026-08-17 — `build_ice_servers` + `.env` REMOTE_TURN_URLS includes `turns:…:443?transport=tcp` + STUN)*
- [x] **C-RD-SMOOTH-5 (1.4.71)** Viewer paints JPEG-WS as video (≥24 fps), not a 12 Hz image. ICE fail keeps the same surface at video rate. *(prod 2026-08-17 — `queueJpegFrame` + `requestAnimationFrame` latest-frame path in dashboard_remote.html)*
- [x] **C-RD-VIEW-1** Software cursor over video/JPEG (hardware cursor not in bitstream).
- [x] **C-RD-VIEW-4 / CL-RD-CAD-1** CAD = `POST /api/remote/cad` → `remote_send_sas` only. No synthetic Ctrl+Alt+Del key.
- [x] **C-RD-VIEW-6** `black_frame` / `winlogon_capture_black` / `CAPTURE_NO_DESKTOP` → degraded banner, not healthy Live.
- [x] **C-RD-VIEW-7 / C-RD-5** ICE fail → JPEG-WS ≤2s on the **same** surface; clear connected UI.
- [x] **C-RD-VIEW-8** Show `stream_progress` (`running` → `capturing` → `ws`/`webrtc` → `live` / `switching` / `failed`).
- [x] **C-RD-1** Offer WebRTC only if `capabilities.webrtc.available` and `"webrtc"∈transports`.
- [x] **C-RD-2** While WebRTC connected, do not also paint JPEG.
- [x] **CL-RD-S0-1/2** Path B (lock row) omits `session_id` and `username`.
- [x] **C-RD-CON-8** `list_sessions` / dashboard always expose Logon/Lock sibling (`pre_logon`) even if the agent omitted it.
- [x] **CL-RD-S0-5 / C-RD-P0-WL-2** `SESSION0_HELPER_SPAWN_FAILED` / sub-1500 B JPEG / `black_frame` never paint a healthy Live badge.
- [x] **C-RD-VIEW-8 / FOLLOW-4** Honor `phase=switching`; Live may drop to switching/degraded; frozen frames &gt;2s drop Live.
- [x] **C-RD-FOLLOW-9/10 (1.4.70)** Same `stream_id` on **lock/logoff** as on unlock. Do **not** send a second Start or auto-pick the user. Do **not** treat `list_sessions` username as “Default is live”. *(prod 2026-08-17 — armed follow keeps `console`/`wl`; second Start blocked while `currentStreamId` set; Bağlan→Durdur single start)*
- [x] **C-RD-VIEW-11 (1.4.70)** Default Connect must **not** auto-open “Kullanıcıya bağlan” / password modal over the player. Operator types on the remote surface. *(prod 2026-08-17 — follow/winlogon `selectedNeedsPrepare=false`; modal not shown on Default Connect)*

---

## P0 — Process inspect

SoT: [`../features/process-inspect.md`](../features/process-inspect.md)

- [x] **CL-PROC-INSPECT-1** Whitelist `inspect_process` on control WS (same catalog as `list_processes`).
- [x] **CL-PROC-INSPECT-2** “?” / İncele dispatches immediately (no extra modal beyond evidence).
- [x] **CL-PROC-INSPECT-6** **No** “Onayla ve Gönder” / required `confirm:true`. Inspect is not destructive.
- [x] Rundll32 `dll,Entry` is **not** shown as lolbin. lolbin only: `http` / `javascript:` / UNC.

---

## P0 — Self-update

SoT: [`../features/self-update.md`](../features/self-update.md)

- [x] **CL-UPD-TRUST-1/2** `signed=null` / `checksum_valid=null` / “trust pending” = observe. Must **not** block download. Missing Authenticode must not abort.
- [x] **CL-UPD-ASSET-1/2** GitHub Release binary name: **only** `asteria-client-installer.exe`. `installer_name` + `download_url` match. Do not publish `cloud-client-installer.exe`.
- [x] **C-UPD-PROG** Surface download ticks (`phase`, `progress_pct`). Silent `running` &gt;5s without a tick = fail UI.
- [x] `check_update` is inspect-only (does not install).

---

## P0 — Threat intel ACK

SoT: [`../features/threat-intel.md`](../features/threat-intel.md)

- [x] Agent GET 200 → ACK expected. Agent GET **304** → **no ACK** (do not treat missing ACK as agent bug).
- [x] ACK `stats.firewall_current` = standing `AR-INTEL-*` count (optional field; display if present).
- [x] Client does **not** fetch Abuse.ch / CISA. Ingest stays cloud-only ([`threat-intel-ingest.md`](./threat-intel-ingest.md) appendix).

---

## Already shipped (keep `[x]`; re-open only if regression)

- [x] **CL-BRICK-3** Dashboard auto-link (no steal)
- [x] **CL-BRICK-5** Undo email + one-time key → `enable_account`
- [x] **CL-BRICK-STATUS** `undo_mail_path` on account-status
- [x] **CL-UNLINK-MAIL** Unlink magic-link; direct unlink → `missing_confirm_code`
- [x] C-USER users page + confirm on create / remote_logon / reboot (see [`../features/server-management.md`](../features/server-management.md))
- [x] Tools Repair whitelist + `/dashboard/server/tools`

---

## P1 / ops (do not rush)

- [x] **CL-BRICK-DEFAULTS** New threat_config: silent-hours auto flags **OFF**
- [x] **CL-CANARY** `ASTERIA_FLEET_CANARY_*` masters default **0**; emit `fleet_rollout{}` when verified in prod
- [x] **CL-ENV-V2** Dual-read envelope v2; **no** production `version:2` emit
- [x] **CL-OFFLINE** Offline urgent queue one-host pilot (client flag still off)
- [x] **CL-SUNSET** Dual-brand / legacy HMAC toward **2026-10-01** ([`PRODUCT_BRANDING.md`](./PRODUCT_BRANDING.md))

---

## Explicitly not this sprint

- Envelope **enforce** / operator key custody
- `isolate_host` full path
- Removing `yesnext-*` verify before sunset date
- PM2 / nginx / dashboard HTML in this repo

---

## After you ship

1. Cloud RD P0 boxes for **1.4.70/71** (VIEW-9, SMOOTH-1/2/5, FOLLOW-9/10, VIEW-11) ticked **2026-08-17** on `asteria.run` (agent pin **≥4.9.103**).
2. Pull `asteria-contract` on the cloud host and `publish_contract.sh`.
3. **Client:** tick [`../features/CLIENT_CHECKLIST.md`](../features/CLIENT_CHECKLIST.md) only after Derin-Web (or lab) Runs A–F PASS on **≥4.9.103**.
