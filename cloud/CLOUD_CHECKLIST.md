# Cloud / dashboard checklist

> Contract **≥ 1.4.66**. Cloud this file, tick `[x]`, PR note with commit/date.
> Do **not** add new SoT markdown here — implement against `features/*`.
>
> Pin: client **≥ 4.9.96** recommended · RD topology **≥ 4.9.95** · inspect **≥ 4.9.93**
> API: `https://asteria.run` · HMAC `asteria-chp-v1`

**Read first:** [`../features/README.md`](../features/README.md)  
**Shared wire:** [`../api/01-auth.md`](../api/01-auth.md) · [`../api/03-control-websocket.md`](../api/03-control-websocket.md)

How to mark: `[x]` = verified in **production** dashboard/API (not just code review).

---

## P0 — Remote Desktop

SoT: [`../features/remote-desktop.md`](../features/remote-desktop.md)

- [x] **C-RD-TOPO-1** Default Connect / “Logon · varsayılan” Start: `{ topology:"follow", stream_id, fps }` only. Do **not** send `prefer`, `pre_logon`, `desktop`, `session_id`, `username`.
- [x] **C-RD-TOPO-2** Lock / Logon **row**: `topology=winlogon` + `prefer=winlogon` + `pre_logon=true` + `desktop=Winlogon`. Omit SID and username.
- [x] **C-RD-TOPO-3** User shortcut: `session_id` + `username`. Never auto-pick first Active SID.
- [x] **C-RD-TOPO-4** After Enter: **same `stream_id`**, no second Start, no “pick administrator”.
- [x] **C-RD-TOPO-5** Warn agent &lt;4.9.26; recommend **≥4.9.95**. 4.9.94 follow-skip is not acceptance.
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

1. Tick boxes above in a contract PR **or** reply with ID list + dashboard commit.
2. Pull `asteria-contract` on the cloud host and `publish_contract.sh`.
3. Lab: Default Connect on a **logged-on** host must not spawn Winlogon / jpeg=0B (`SESSION0_HELPER_SPAWN_FAILED`).
