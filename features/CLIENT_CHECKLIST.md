# Client / agent checklist

> Contract **≥ 1.4.68**. Windows agent this file. Do **not** add new SoT here —
> implement against the linked `features/*` file.
>
> Pin: **≥ 4.9.96** recommended · RD topology **≥ 4.9.95** · inspect **≥ 4.9.93**.
> **4.9.94 follow-skip is not acceptance.**
>
> **2026-08-15:** agent **4.9.96** on GitHub. Unit suite for this checklist: 95
> passed. Inspect / update / intel / wire / CON-8 / S0 / CAD ticked.
> **Open:** live Default Connect pixels (TOPO-1/2/FOLLOW) and ICE-fail JPEG on a
> logged-on lab host talking to production `topology=follow`.

**Read first:** [`README.md`](./README.md)  
**Cloud ticks:** [`../cloud/CLOUD_CHECKLIST.md`](../cloud/CLOUD_CHECKLIST.md)

How to mark remaining RD boxes: `[x]` only after a real console lab.

---

## P0 — Remote Desktop

SoT: [`remote-desktop.md`](./remote-desktop.md)

Lab host must be **logged on** at the physical/console session for Default Connect.

- [ ] **C-RD-TOPO-1** Honor `{ topology:"follow", stream_id, fps }` with **no** `prefer`/`pre_logon`/`desktop`. Capture **WinSta0\\Default** via DXGI/NVENC. Do **not** spawn Winlogon helper. jpeg&gt;0. `SESSION0_HELPER_SPAWN_FAILED` on this path = FAIL. *(shipped 4.9.95/96 — live lab still open)*
- [ ] **C-RD-TOPO-2** Honor lock-row `{ topology:"winlogon", prefer:"winlogon", pre_logon:true, desktop:"Winlogon" }` with **no** SID/username. Helper is legitimate. Pixels = LogonUI/Winlogon, not that user’s wallpaper. *(shipped — live lab still open)*
- [ ] **C-RD-TOPO-4 / FOLLOW** After Enter/unlock: **same `stream_id`**, tear down Winlogon helper, follow console SID onto Default **before** the next frame. Do not freeze last Logon JPEG &gt;2s (`phase=switching` then `live`). Dual-write Winlogon+Default input forbidden. *(shipped — live lab still open)*
- [x] **C-RD-CON-8** Every `list_sessions` result includes a Logon/Lock sibling `pre_logon:true` (no invented SID `1`).
- [x] **C-RD-S0 / honesty** Path B helper in the **interactive** session (`CreateProcessAsUser` + `lpDesktop=winsta0\\Winlogon`). jpeg≈0B → `SESSION0_HELPER_SPAWN_FAILED` (`streaming:false`) unless TOPO-1 live Default skip. `black_frame` / `gdi+black` honest in `t:meta`. *(accepted 4.9.84)*
- [x] **CAD / input** `remote_send_sas` only (no typed Ctrl+Alt+Del). Meta `inputs_applied` / `last_input_event` live. Input targets the **same** desktop as capture. *(accepted 4.9.86)*
- [x] **WebRTC** Advertise `capabilities.webrtc` only if real. *(unit)* ICE fail → JPEG-WS on the same stream; do not keep a zombie `connected`. *(ICE live lab still open)*

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
