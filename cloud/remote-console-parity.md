# Console Remote Desktop — physical-console parity (Winlogon)

> **Contract VERSION:** **1.4.59** (viewer C-RD-VIEW since **1.4.47**; named topology **1.4.59**)  
> Status: **Normative (C-RD-TOPO ≥4.9.95 · Session-0 pixels ≥ 4.9.84 · follow after logon ≥ 4.9.93 · dashboard C-RD-VIEW ≥ 1.4.47)**  
> Related: [`REMOTE_DESKTOP_WINLOGON.md`](./REMOTE_DESKTOP_WINLOGON.md) ·
> [`../agent/remote-desktop-p0.md`](../agent/remote-desktop-p0.md) ·
> [`../agent/winlogon-session0-capture.md`](../agent/winlogon-session0-capture.md) ·
> [`../api/05-remote-desktop.md`](../api/05-remote-desktop.md) ·
> [`../agent/remote-input.md`](../agent/remote-input.md) ·
> [`../agent/remote-stream-progress.md`](../agent/remote-stream-progress.md) ·
> [`REMOTE_DESKTOP_SMOOTHNESS.md`](./REMOTE_DESKTOP_SMOOTHNESS.md)

## Goal

Dashboard **Logon ekranı** must feel like sitting at the server
(Proxmox VE Konsol / Chrome Remote Desktop to a locked host):

| Operator action | Expected |
|-----------------|----------|
| Pick **Logon ekranı** → Connect | See Windows logon / lock UI pixels (not black) |
| See mouse | **Visible pointer** over the frame (software cursor — see C-RD-VIEW-*) |
| Type username/password on stream | Keys go to Winlogon secure desktop |
| CAD | `remote_send_sas` → Secure Attention Sequence on that session |
| After successful logon | Stream follows transition to Default desktop |
| Mouse clicks | Land on Winlogon controls |

This is **not** a hypervisor framebuffer and **does not require noVNC/RFB**.
Agent runs as SYSTEM (often Session 0) and MUST attach the **interactive console
session's `WinSta0\Winlogon` desktop**, capture real pixels, and inject input there.
Transport remains **WebRTC and/or JPEG-over-WS** (existing Asteria remote wire).

---

## Named Start topologies (C-RD-TOPO-*) — **P0 · 1.4.59 · agent ≥4.9.95**

Lab **4.9.94** follow-skip is **incomplete** (legacy `prefer=winlogon` still on
default Connect). Target agent **≥4.9.95**. Cloud MUST send **one** of A/B/C —
never a single Start shape for every Connect.

| ID | Rule |
|----|------|
| **C-RD-TOPO-1** | Default Connect / “Logon · varsayılan”: `params.topology = "follow"`. **Do not send** `prefer`, `pre_logon`, `desktop`, `session_id`, `username`. Lab 4.9.93 FAIL: `prefer=winlogon` → Winlogon helper + `SESSION0_HELPER_SPAWN_FAILED` + jpeg=0B while console user already Active. Follow uses DXGI/NVENC Default. |
| **C-RD-TOPO-2** | Logon / Lock **row** (empty host, lock, SAS): `topology = "winlogon"` + `prefer: "winlogon"` + `pre_logon: true` + `desktop: "Winlogon"`. Omit `session_id` and `username`. Winlogon helper is legitimate here. |
| **C-RD-TOPO-3** | User shortcut: `session_id` + `username`. Do **not** auto-select the first Active SID. |
| **C-RD-TOPO-4** | After Enter: **same `stream_id`**, no second Start, no “pick administrator”. Follow `WTSGetActiveConsoleSessionId` → `WinSta0\Default` (C-RD-FOLLOW). Winlogon spawn while Default is live must not be a terminal FAIL. |
| **C-RD-TOPO-5** | Min agent for this wire: **≥4.9.95**. Warn below 4.9.26; 4.9.94 is not acceptance. |

**A — Default Connect**

```json
{
  "command_type": "remote_stream_start",
  "params": { "topology": "follow", "stream_id": "…", "fps": 12 }
}
```

**B — Logon / Lock row**

```json
{
  "command_type": "remote_stream_start",
  "params": {
    "topology": "winlogon",
    "prefer": "winlogon",
    "pre_logon": true,
    "desktop": "Winlogon",
    "stream_id": "…",
    "fps": 12, "quality": 40, "max_width": 1280
  }
}
```

Cloud ≥**1.4.50** **omits** `session_id` on A and B (CL-RD-S0). Optional SID only for
path C (user/session rows).

## Start wire (C-RD-CON-* still apply on path B)

| ID | Rule |
|----|------|
| **C-RD-CON-1** | Honor `prefer=winlogon` / `pre_logon` / `desktop=Winlogon` even when another user session is Active (sibling lock row) |
| **C-RD-CON-2** | If `session_id` omitted, resolve with `WTSGetActiveConsoleSessionId` (or equivalent) — **do not assume SID 1**. Session-0 spawn: [`../agent/winlogon-session0-capture.md`](../agent/winlogon-session0-capture.md) |
| **C-RD-CON-3** | **Never** bind username on this path (cloud strips it). Username forces Default desktop stick |
| **C-RD-CON-4** | Capture named desktop `Winlogon` first; `OpenInputDesktop` alone is insufficient when Default is active |
| **C-RD-CON-5** | Persistent `capture_method=gdi+black` / all-black frames while claiming `desktop=Winlogon` = **FAIL** (see P0-A) |
| **C-RD-CON-6** | After credentials accepted, switch capture **and input** to Default for that console session **without** a second Start (see **C-RD-FOLLOW-***)
| **C-RD-CON-7** | `remote_send_sas` must target the same console/session as the stream (no username on Winlogon CAD) |
| **C-RD-CON-8** | Health / `list_sessions` always emit a **Logon / Lock** row with `pre_logon:true` (+ `can_capture` when known), including as sibling of an Active user on the same SID |
| **C-RD-CON-9** | Surface progress honestly: `stream_progress` stages + capture meta (`desktop`, `capture_method`, `black_frame`) |

---

## Dashboard viewer MUST (C-RD-VIEW-*) — Proxmox-like cursor and controls

DXGI/GDI capture typically **does not bake the hardware mouse cursor** into the
JPEG/H.264 bitstream. Proxmox noVNC draws a local cursor; Asteria dashboard MUST
do the same on the existing viewer surface.

| ID | Rule |
|----|------|
| **C-RD-VIEW-1** | Draw a **local software cursor** over `<video>` / JPEG `<img>` at the last pointer position (Proxmox-style). Do not rely on frames containing the OS cursor |
| **C-RD-VIEW-2** | Map pointer with normalized `x,y ∈ [0,1]` against `meta.native_width/height` + `origin_x/y` ([`05-remote-desktop.md`](../api/05-remote-desktop.md) §2). Support negative origins |
| **C-RD-VIEW-3** | Coalesce only `move`; flush `mousedown` / `mouseup` / `wheel` / `key*` immediately (C-RD-4) |
| **C-RD-VIEW-4** | Toolbar **CAD** → `remote_send_sas` / `POST /api/remote/cad` **only** — never Ctrl+Alt+Delete as typed keys |
| **C-RD-VIEW-5** | Default Connect = **C-RD-TOPO-1** (`topology=follow` only). Logon/Lock row = **C-RD-TOPO-2**. |
| **C-RD-VIEW-6** | On `black_frame` / `winlogon_capture_black` / `CAPTURE_NO_DESKTOP`: explicit degraded banner — never a silent empty player |
| **C-RD-VIEW-7** | Prefer WebRTC when advertised; ICE fail → JPEG-WS ≤2s on the **same** surface |
| **C-RD-VIEW-8** | Show `stream_progress` (`running` → `capturing` → `ws`/`webrtc` → `live`) |
| **C-RD-VIEW-9** | Min client gate: warn if agent &lt; **4.9.26**; recommend ≥ **4.9.95** for named topology + live Default skip (C-RD-FOLLOW) |
| **C-RD-VIEW-10** | Do **not** auto-select the first Active user. Default option = Logon. Frozen frames (`diag=agent_ws_no_frames` / `age_sec` high) MUST drop Live badge. |

---

## After logon: follow the console (C-RD-FOLLOW-*) — **P0 · ≥4.9.93**

Chrome Remote Desktop model: one connection to the **physical console**.
The operator types credentials **on the stream**. When Windows creates/activates
the interactive user session, capture and input **stay on that console**.
A user list is optional (direct attach to an already-open session). It is **not**
required to start, and **must not** be required after Enter.

Lab **4.9.92** FAIL: password+Enter → `administrator` SID Active, helper still on
Winlogon, last JPEG frozen, `diag=agent_ws_no_frames`.

| ID | Rule |
|----|------|
| **C-RD-FOLLOW-1** | After interactive logon or unlock, capture **and** input MUST follow `WTSGetActiveConsoleSessionId` onto `WinSta0\Default` of that session. **Same `stream_id`.** No second `remote_stream_start`. Dashboard must not ask the operator to pick the user again. |
| **C-RD-FOLLOW-2** | Do **not** leave the Winlogon helper attached once LogonUI/SAS is dismissed and the console is a user Default desktop. If the helper is session-bound, tear it down and reattach (or spawn) on the new SID. |
| **C-RD-FOLLOW-3** | Handle `WTS_CONSOLE_CONNECT`, `WTS_SESSION_LOGON`, `WTS_SESSION_UNLOCK`, and secure-desktop → Default switches: `SetThreadDesktop` / helper rebind **before** the next frame. |
| **C-RD-FOLLOW-4** | Across the switch, frames MUST NOT freeze on the last Logon JPEG for &gt;2s. Emit `stream_progress.phase=switching` then `live`. A growing `age_sec` with `agent_ws_no_frames` after a successful logon is **FAIL**. |
| **C-RD-FOLLOW-5** | After Default attach, prefer DXGI / NVENC (or the normal interactive-session encoder). Winlogon JPEG helper (~4 fps) is **not** the post-logon path. |
| **C-RD-FOLLOW-6** | `remote_stream_start` **without** `session_id` = console **follow** (`topology=follow`). Winlogon helper only while LogonUI/lock/no user. Do **not** bind the first Active SID from `list_sessions`. Optional `session_id` / `username` is the shortcut path only. `topology=winlogon` is the lock/logon row. |
| **C-RD-FOLLOW-7** | Input (key / mouse / CAD) targets the **same** desktop as capture after the switch. Dual-write Winlogon+Default after logon is forbidden (causes 4× keys). |
| **C-RD-FOLLOW-8** | `t:meta` MUST update `desktop`, `session_id`, `username`, `capture_method` after follow (live meta, not start snapshot). |

### Client handoff (paste)

Target **≥4.9.95**. Contract **1.4.59**. Paste: [`CLOUD_HANDOFF_1.4.59.md`](./CLOUD_HANDOFF_1.4.59.md).

Proof on Derin-Web: Logon Start (no user pick) → CAD → type password → Enter → **desktop wallpaper / shell**, `desktop=default`, frames keep moving, `inputs_applied++` still works. Do **not** require Stop → pick administrator → Connect.

---

- Hypervisor / iLO / Proxmox VGA passthrough (out of agent scope)
- Capturing Session 0 service desktop as a substitute for Winlogon
- Auto-typing credentials from cloud (operator types on stream)
- Embedding noVNC / opening TCP :5900 on agents (optional future transport only)

---

## Cloud (shipped ≥ 1.4.43; viewer C-RD-VIEW shipped ≥ 1.4.47; omit SID ≥ **1.4.50**)

- Preserve `pre_logon` / `can_capture` through `normalize_sessions`
- Default Connect: **C-RD-TOPO-1** (`topology=follow` only — no prefer/pre_logon/desktop)
- Logon/Lock row: **C-RD-TOPO-2**
- UI: Winlogon banner + black-frame honesty (`showWinlogonHint`)
- CAD on Winlogon path omits username **and** forced SID
- **1.4.47:** software cursor overlay + full C-RD-VIEW-* on remote console page (**dashboard accepted**)

---

## Acceptance

### Client ≥ 4.9.49 (or train closing C-RD-CON-*)

- [x] Empty host (no user logged on): Logon ekranı → non-black logon UI ≤ 10s — **4.9.84 Derin-Web**
- [ ] User Active + lock: sibling Logon row → lock UI, not that user's Default desktop
- [ ] Type password on stream → unlock/logon succeeds → **Default desktop appears on the same stream** (C-RD-FOLLOW; open through **4.9.92**)
- [ ] CAD while on Winlogon → SAS UI (Task Manager / password change options as applicable) — **open 1.4.52 / client ≥4.9.85** (lab: false SendSAS ok on 4.9.84)
- [x] Omit `session_id` → still attaches correct console (not wrong RDP session / invent SID 1)
- [x] No sustained `gdi+black` while `desktop=Winlogon` in hello/meta — helper `persistent-winlogon-helper:raw`

### Cloud / dashboard

- [x] `pre_logon` preserved in session normalize
- [x] No `session_id=1` hardcode
- [x] Winlogon hint / black-frame UX wired
- [x] **C-RD-VIEW-1** software cursor (`rdSoftCursor` + `cursor:none`) — shipped with **1.4.47** viewer
- [x] **C-RD-VIEW-2..3** pointer [0,1] content-box; move coalesce; critical flush
- [x] **C-RD-VIEW-4..5** CAD → `/api/remote/cad` only; Logon Start wire (no username, `max_width:1280`)
- [x] **C-RD-VIEW-6** black_frame / Winlogon black / `CAPTURE_NO_DESKTOP` degraded banner
- [x] **C-RD-VIEW-7** ICE fail → JPEG ≤2s, same surface
- [x] **C-RD-VIEW-8** `stream_progress` stages on existing pipe
- [x] **C-RD-VIEW-9** agent &lt;4.9.26 warn / ≥4.9.93 follow recommend
- [x] **C-RD-VIEW-10** default option Logon; no auto-pick Active user; stale Live dropped

### Remaining (client lab — not dashboard)

**C-RD-FOLLOW** after Enter: **≥4.9.93**. 4.9.92 stays on Winlogon helper → frozen JPEG.
---

## Min client

**≥ 4.9.49** wire; **≥ 4.9.84** Session-0 Logon pixels; **≥ 4.9.93** C-RD-FOLLOW (logon → Default, same stream).
