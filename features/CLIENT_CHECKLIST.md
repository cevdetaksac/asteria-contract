# Client / agent checklist

> Contract **≥ 1.4.67**. Windows agent this file, tick `[x]` on a **lab host**
> talking to `https://asteria.run`. Do **not** add new SoT here — implement
> against the linked `features/*` file.
>
> Pin: **≥ 4.9.96** recommended (intel 304 + installer name) · RD topology **≥ 4.9.95**
> · inspect **≥ 4.9.93**. **4.9.94 follow-skip is not acceptance.**

**Read first:** [`README.md`](./README.md)  
**Cloud ticks (already done on this VERSION):** [`../cloud/CLOUD_CHECKLIST.md`](../cloud/CLOUD_CHECKLIST.md)

How to mark: `[x]` = verified on a real Windows host (not code review).

---

## P0 — Remote Desktop

SoT: [`remote-desktop.md`](./remote-desktop.md)

Lab host must be **logged on** at the physical/console session for Default Connect.

- [ ] **C-RD-TOPO-1** Honor `{ topology:"follow", stream_id, fps }` with **no** `prefer`/`pre_logon`/`desktop`. Capture **WinSta0\\Default** via DXGI/NVENC. Do **not** spawn Winlogon helper. jpeg&gt;0. `SESSION0_HELPER_SPAWN_FAILED` on this path = FAIL.
- [ ] **C-RD-TOPO-2** Honor lock-row `{ topology:"winlogon", prefer:"winlogon", pre_logon:true, desktop:"Winlogon" }` with **no** SID/username. Helper is legitimate. Pixels = LogonUI/Winlogon, not that user’s wallpaper.
- [ ] **C-RD-TOPO-4 / FOLLOW** After Enter/unlock: **same `stream_id`**, tear down Winlogon helper, follow console SID onto Default **before** the next frame. Do not freeze last Logon JPEG &gt;2s (`phase=switching` then `live`). Dual-write Winlogon+Default input forbidden.
- [ ] **C-RD-CON-8** Every `list_sessions` result includes a Logon/Lock sibling `pre_logon:true` (no invented SID `1`).
- [ ] **C-RD-S0 / honesty** Path B helper in the **interactive** session (`CreateProcessAsUser` + `lpDesktop=winsta0\\Winlogon`). jpeg≈0B → `SESSION0_HELPER_SPAWN_FAILED` (`streaming:false`) unless TOPO-1 live Default skip. `black_frame` / `gdi+black` honest in `t:meta`.
- [ ] **CAD / input** `remote_send_sas` only (no typed Ctrl+Alt+Del). Meta `inputs_applied` / `last_input_event` live. Input targets the **same** desktop as capture.
- [ ] **WebRTC** Advertise `capabilities.webrtc` only if real. ICE fail → JPEG-WS on the same stream; do not keep a zombie `connected`.

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
