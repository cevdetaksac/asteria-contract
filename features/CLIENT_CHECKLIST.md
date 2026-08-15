# Client / agent checklist

> Contract **≥ 1.4.68**. Windows agent this file, tick `[x]` on a **lab host**
> talking to `https://asteria.run`. Do **not** add new SoT here — implement
> against the linked `features/*` file.
>
> Pin: **≥ 4.9.97** for RD **pixels** (C-RD-PIX) · topology names **≥ 4.9.95**
> · intel+installer **≥ 4.9.96** · inspect **≥ 4.9.93**.
> **4.9.94 follow-skip is not acceptance.** **gdi+black / solid-black JPEG is not acceptance.**

**Read first:** [`README.md`](./README.md) · RD SoT: [`remote-desktop.md`](./remote-desktop.md)  
**Cloud ticks (already done):** [`../cloud/CLOUD_CHECKLIST.md`](../cloud/CLOUD_CHECKLIST.md)

How to mark: `[x]` = verified on a real Windows host (not code review).
2026-08-15 Derin-Web dashboard: Default Connect → viewer **“Görüntü tam değil”**,
meta `persistent-user-helper` + `gdi+black`. **PIX boxes below are OPEN.**

---

## P0 — Remote Desktop

SoT: [`remote-desktop.md`](./remote-desktop.md)

Do **Run A and Run C separately** (empty host vs logged-on). Mixing them hides bugs.

### Capture pixels (C-RD-PIX) — why the dashboard is black

- [ ] **C-RD-PIX-1** Healthy frame ≠ JPEG bytes. Solid 1024×768 black is FAIL. Need `black_frame=false` plus chrome (variance / bright_ratio / LogonUI hwnd or DXGI wallpaper).
- [ ] **C-RD-PIX-2** `desktop=Winlogon` → LogonUI/SAS pixels ≤3s. Else `winlogon_capture_black` / `black_frame:true` and **not** Live / not fake `streaming:true`.
- [ ] **C-RD-PIX-3** Empty host + `topology=follow`: interactive Winlogon helper (`CreateProcessAsUser` + `lpDesktop=winsta0\\Winlogon`). **FAIL** if `persistent-user-helper` + `gdi+black` (lab 2026-08-15).
- [ ] **C-RD-PIX-4** Logged-on console + `topology=follow`: DXGI/NVENC Default only. No Winlogon helper. No `SESSION0_HELPER_SPAWN_FAILED`.
- [ ] **C-RD-PIX-5** Honest `capture_method` (`dxgi+nvenc` / `persistent-winlogon-helper:raw` / `gdi+black`). `gdi+black` is never success.
- [ ] **C-RD-PIX-6** WebRTC `connected` / JPEG-suppress only **after** one healthy frame. Black + nvenc = FAIL.
- [ ] **C-RD-PIX-7** Live `t:meta` ≤5 frames: desktop, method, black_frame, variance, bright_ratio, logonui_hwnd_count, session_id, username.

### Topology / follow / S0 / CAD (still required)

- [ ] **C-RD-TOPO-1** Honor `{ topology:"follow", stream_id, fps }` with **no** `prefer`/`pre_logon`/`desktop`/SID. Combine with PIX-3 or PIX-4 depending on console state.
- [ ] **C-RD-TOPO-2** Lock row `{ topology:"winlogon", prefer:"winlogon", pre_logon:true, desktop:"Winlogon" }`, no SID. Pixels = LogonUI, not wallpaper.
- [ ] **C-RD-TOPO-4 / FOLLOW** After Enter/unlock: **same `stream_id`**, tear down helper, Default ≤2s (`phase=switching` then `live`). No dual-write input.
- [ ] **C-RD-CON-8** Every `list_sessions` includes Logon/Lock sibling `pre_logon:true` (no SID `1`).
- [ ] **C-RD-S0** Path B helper in the **interactive** session. jpeg≈0B → `SESSION0_HELPER_SPAWN_FAILED` (`streaming:false`) unless TOPO-1 **logged-on** DXGI skip.
- [ ] **CAD / input** `remote_send_sas` only. Meta `inputs_applied` / `last_input_event` live. Input desktop = capture desktop.
- [ ] **WebRTC** Advertise only if real. ICE fail → JPEG-WS; no zombie `connected`.

### Lab commands (copy)

Empty host, Default Connect:

```text
1. Log off / lock the console so LogonUI is on the physical screen.
2. Open /dashboard/remote?token=… — wait idle (Bağlan visible).
3. Leave target = Logon · varsayılan. Click Bağlan.
4. PASS: password box or lock chrome in the viewer ≤3s.
5. FAIL: black player + “Görüntü tam değil” + capture_method=gdi+black / persistent-user-helper.
```

Logged-on Default Connect: log on locally first, then Bağlan — must match the physical desktop (DXGI), not Winlogon.

---

## P0 — Process inspect

SoT: [`process-inspect.md`](./process-inspect.md)

- [ ] **C-PROC-INSPECT-1** Health `top_processes[]` stays lean; `inspectable:true` when PID inspectable.
- [ ] **C-PROC-INSPECT-2** `inspect_process {pid}` returns cmdline, parent, rundll32 dll/export, peers, `verdict`. Not destructive.
- [ ] **C-PROC-INSPECT-3** `rundll32` + `foo.dll,Entry` is **not** `lolbin`. lolbin only: `http` / `javascript:` / UNC.
- [ ] Do **not** locally require `confirm:true` for inspect.

---

## P0 — Self-update

SoT: [`self-update.md`](./self-update.md)

- [ ] **CL-UPD-ASSET** Download **only** `asteria-client-installer.exe`. Rewrite legacy `cloud-client-installer.exe` in URL/name.
- [ ] **CL-UPD-TRUST** `signed=null` / missing Authenticode = observe; download continues.
- [ ] **C-UPD-PROG-1/2** While `running`, emit `phase` + `progress_pct` (and bytes) ≥ every 2s. Silent &gt;5s = FAIL.
- [ ] `check_update` inspect-only (no install). Single-flight overlapping Update now.

---

## P0 — Threat intel

SoT: [`threat-intel.md`](./threat-intel.md)

- [ ] GET **200** → apply + **ACK** (include `stats.firewall_current` if you count standing `AR-INTEL-*`).
- [ ] GET **304** → keep cache, expire/orphan reconcile, **no ACK**.
- [ ] Do **not** fetch Abuse.ch / CISA / ThreatFox. Cloud ingest only.
- [ ] Apply rules as `AR-INTEL-*` (not `AR-BLOCK-*`). WS `threat_intel_updated` → GET immediately.

---

## Shared wire (regression)

- [ ] Agent HTTP/WS: `Authorization: Bearer` only (no `?token=`).
- [ ] Command HMAC verify `asteria-chp-v1` (legacy `yesnext-chp-v1` until 2026-10-01).
- [ ] Honor `fleet_rollout.gates` — if gate is false, do not apply silent-hours auto / NG auto-contain / isolate / offline-urgent even if local config is on.
- [ ] `security.offline_urgent_queue` stays **default off**.

---

## Explicitly not this sprint (client)

- Envelope v2 **enforce** (observe-only if you parse `version:2`)
- `isolate_host` full path
- Dropping `yesnext-chp-v1` verify before sunset
