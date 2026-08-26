# Client / agent checklist

> Contract **≥ 1.4.77**. Windows agent this file. Do **not** add new SoT here —
> implement against the linked `features/*` file.
>
> Pin: **≥ 4.9.110** RD JPEG-WS primary + honest capture · **≥ 4.9.109** dual-channel Attacks ·
> **≥ 4.9.108** RDP NLA vs NETWORK:445 ·
> **≥ 4.9.107** RD post-logon follow + capture_diag ·
> topology **≥ 4.9.95** · intel+installer **≥ 4.9.96** · inspect **≥ 4.9.93**.
> **4.9.94 follow-skip is not acceptance.**
> **4.9.99–4.9.109 Derin-Web `gdi+black` / `persistent-user-helper` / `dxgi:pending` is not acceptance.**
> **Frozen Welcome after password without Default follow is not acceptance.**
> **Version string alone is not acceptance — console lab required.**
>
> **2026-08-26:** agent **4.9.110** ships JPEG-WS primary video + lock probe honesty;
> **4.9.109** dual-channel Attacks; **4.9.107** FOLLOW unlock + `t:capture_diag`.
> **Open ticks = console lab only** (Derin-Web / Ninety-Web Run A–F on **≥4.9.110**).
> Do **not** `[x]` PIX/TOPO/FOLLOW from unit alone.

**Read first:** [`README.md`](./README.md) · RD SoT: [`remote-desktop.md`](./remote-desktop.md)  
**Cloud ticks:** [`../cloud/CLOUD_CHECKLIST.md`](../cloud/CLOUD_CHECKLIST.md)

How to mark PIX/TOPO: `[x]` only after a real console lab (not unit-only).

---

## P0 — Remote Desktop

SoT: [`remote-desktop.md`](./remote-desktop.md)

Do **Run A and Run C separately** (empty/lock vs logged-on unlocked). Mixing them hides bugs.

### Capture pixels (C-RD-PIX) — why the dashboard is black

- [ ] **C-RD-PIX-1** Healthy frame ≠ JPEG bytes. Solid 1024×768 black is FAIL. Need `black_frame=false` plus chrome (variance / bright_ratio / LogonUI hwnd or DXGI wallpaper).
- [ ] **C-RD-PIX-2** `desktop=Winlogon` → LogonUI/SAS pixels ≤3s. Else `winlogon_capture_black` / `black_frame:true` and **not** Live / not fake `streaming:true`.
- [ ] **C-RD-PIX-3** Empty host **or locked console** + `topology=follow`: interactive Winlogon helper (`CreateProcessAsUser` + `lpDesktop=winsta0\\Winlogon` + **winlogon.exe token**). **FAIL** if `persistent-user-helper` + `gdi+black` (lab 4.9.99…**4.9.103**).
- [ ] **C-RD-PIX-4** Logged-on **and unlocked** console + `topology=follow` (or user SID Start): DXGI/NVENC `WinSta0\Default` only. No Winlogon helper. No `SESSION0_HELPER_SPAWN_FAILED`. **FAIL** if username Active but capture stays `persistent-user-helper` black (lab **4.9.103** SID 3). Locked-with-username is PIX-3, not PIX-4.
- [ ] **C-RD-PIX-5** Honest `capture_method` (`dxgi+nvenc` / `persistent-winlogon-helper` / `gdi+black`). Never provisional `dxgi:pending`. `gdi+black` is never success.
- [ ] **C-RD-PIX-6** WebRTC `connected` / JPEG-suppress only **after** one healthy frame. Black + `nvenc` = FAIL.
- [ ] **C-RD-PIX-7** Live `t:meta` ≤5 frames: desktop, method, black_frame, variance, bright_ratio, logonui_hwnd_count, session_id, username.
- [ ] **C-RD-DIAG-1/2** Emit `t:capture_diag` + `meta.capture_diag` on fail/switching/live (helper_token, fail_phase, variance, …).

### Topology / follow / S0 / CAD / smoothness

- [ ] **C-RD-TOPO-1** Honor `{ topology:"follow", stream_id, fps }` with **no** `prefer`/`pre_logon`/`desktop`/SID. Decide PIX-3 vs PIX-4 from **input desktop**, not WTS username. Unknown lock → Winlogon. *(4.9.110 — **live lab still open**)*
- [ ] **C-RD-TOPO-2** Lock row `{ topology:"winlogon", prefer:"winlogon", pre_logon:true, desktop:"Winlogon" }`, no SID. Pixels = LogonUI, not wallpaper.
- [ ] **C-RD-TOPO-4 / FOLLOW-1…11** Enter/unlock: **same `stream_id`**, Default ≤2s (including lock-row Start). Lock/logoff: same stream back to Winlogon chrome. No dual-write input. No second Start. No frozen Welcome.
- [x] **C-RD-CON-8** Every `list_sessions` includes Logon/Lock sibling `pre_logon:true` (no SID `1`).
- [x] **C-RD-S0** Path B helper in the **interactive** session. jpeg≈0B → `SESSION0_HELPER_SPAWN_FAILED` (`streaming:false`) unless TOPO-1 **logged-on** DXGI skip. *(accepted 4.9.84 — must not regress to gdi+black)*
- [x] **CAD / input** `remote_send_sas` only. Meta `inputs_applied` / `last_input_event` live. Input desktop = capture desktop. WS `t:input` immediate (HTTP poll backup only). *(accepted 4.9.86)*
- [ ] **WebRTC / PIX-6 live** Advertise only if real. ICE fail → JPEG-WS of **chrome**; no zombie `connected`. Black+nvenc = FAIL.
- [ ] **C-RD-SMOOTH-1/3/4/7 live** Honor cloud Start **≥30 / ≥72 / 1920**. JPEG-WS primary ≥24 fps while ICE negotiates (`prefer_raw` only after media ready). Lab **4.9.103** showed ~8/Q40 — retest on **≥4.9.110**.

### Lab commands (copy)

Empty / lock, Default Connect (Run A):

```text
1. Log off / lock the console so LogonUI is on the physical screen.
2. Open https://asteria.run/dashboard/remote?token=… — wait idle (Bağlan).
3. Leave target = Logon · varsayılan. Click Bağlan.
4. PASS: password box or lock chrome in the viewer ≤3s; black_frame=false;
   capture_method ≠ gdi+black / persistent-user-helper.
5. FAIL: black player + “görüntü siyah” / gdi+black (seen on 4.9.99–4.9.103).
```

Logged-on unlocked Default Connect (Run C):

```text
1. Locally unlock so Default desktop is on the physical screen.
2. Bağlan on Logon · varsayılan (or administrator · Console).
3. PASS: DXGI/NVENC of that wallpaper/shell; black_frame=false.
4. FAIL: persistent-user-helper + black while username is Active (4.9.103 lab).
```

---

## P0 — Attack classification (4625 / Attacks UI)

SoT: [`threat-engine.md`](./threat-engine.md) · [`../agent/attacks-and-services.md`](../agent/attacks-and-services.md)

- [ ] **C-ATK-1** NLA RDP fail → `/api/attack` **`RDP` / real port / `<failed_logon>`** (not `NETWORK` / `0`)
- [ ] **C-ATK-2** SMB/NtLmSsp type-3 → **`NETWORK` / `445`**
- [ ] **C-ATK-3** EventLog enrich: `logon_type`, `auth_package`, `logon_process`, status/substatus, workstation, `source=eventlog`
- [ ] **C-ATK-4** Bait credential separate (`source=honeypot`); tunnels stopped → no bait rows; **real** fails still report
- [ ] **C-ATK-5** Anonymous / empty Network fails do not flood Attacks
- [ ] **C-ATK-6** OpenSSH Failed/Invalid → **SSH** with honeypot stopped
- [ ] **C-ATK-7** MySQL Access denied (error log) → **MYSQL** with honeypot stopped
- [ ] **C-ATK-9** IIS FTP W3C status 530 → **FTP** with honeypot stopped

---

## P0 — Process inspect

SoT: [`process-inspect.md`](./process-inspect.md)

- [x] **C-PROC-INSPECT-1** Health `top_processes[]` stays lean; `inspectable:true` when PID inspectable.
- [x] **C-PROC-INSPECT-2** `inspect_process {pid}` returns cmdline, parent, rundll32 dll/export, peers, `verdict`. Not destructive.
- [x] **C-PROC-INSPECT-3** `rundll32` + `foo.dll,Entry` is **not** `lolbin`. lolbin only: `http` / `javascript:` / UNC.
- [x] Do **not** locally require `confirm:true` for inspect.

---

## P0 — Self-update

SoT: [`self-update.md`](./self-update.md)

- [x] **CL-UPD-ASSET** Download **only** `asteria-client-installer.exe`. Rewrite legacy `cloud-client-installer.exe` in URL/name.
- [x] **CL-UPD-TRUST** `signed=null` / missing Authenticode = observe; download continues.
- [x] **C-UPD-PROG-1/2** While `running`, emit `phase` + `progress_pct` (and bytes) ≥ every 2s. Silent &gt;5s = FAIL.
- [x] `check_update` inspect-only (no install). Single-flight overlapping Update now.

---

## P0 — Threat intel

SoT: [`threat-intel.md`](./threat-intel.md)

- [x] GET **200** → apply + **ACK** (include `stats.firewall_current` if you count standing `AR-INTEL-*`).
- [x] GET **304** → keep cache, expire/orphan reconcile, **no ACK**.
- [x] Do **not** fetch Abuse.ch / CISA / ThreatFox. Cloud ingest only.
- [x] Apply rules as `AR-INTEL-*` (not `AR-BLOCK-*`). WS `threat_intel_updated` → GET immediately.

---

## Shared wire (regression)

- [x] Agent HTTP/WS: `Authorization: Bearer` only (no `?token=`).
- [x] Command HMAC verify `asteria-chp-v1` (legacy `yesnext-chp-v1` until 2026-10-01).
- [x] Honor `fleet_rollout.gates` — if gate is false, do not apply silent-hours auto / NG auto-contain / isolate / offline-urgent even if local config is on.
- [x] `security.offline_urgent_queue` stays **default off**.

---

## Explicitly not this sprint (client)

- Envelope v2 **enforce** (observe-only if you parse `version:2`)
- `isolate_host` full path
- Dropping `yesnext-chp-v1` verify before sunset
