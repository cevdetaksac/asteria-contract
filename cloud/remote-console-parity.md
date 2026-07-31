# Console Remote Desktop — physical-console parity (Winlogon)

> **Contract VERSION:** **1.4.47**  
> Status: **Normative (client ≥ 4.9.49 · dashboard C-RD-VIEW ≥ 1.4.47)**  
> Related: [`REMOTE_DESKTOP_WINLOGON.md`](./REMOTE_DESKTOP_WINLOGON.md) ·
> [`../agent/remote-desktop-p0.md`](../agent/remote-desktop-p0.md) ·
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

## Start wire (unchanged shape, stricter rules)

```json
{
  "command_type": "remote_stream_start",
  "params": {
    "prefer": "winlogon",
    "pre_logon": true,
    "desktop": "Winlogon",
    "session_id": 2,
    "stream_id": "…",
    "fps": 12, "quality": 40, "max_width": 1280
  }
}
```

| ID | Rule |
|----|------|
| **C-RD-CON-1** | Honor `prefer=winlogon` / `pre_logon` / `desktop=Winlogon` even when another user session is Active (sibling lock row) |
| **C-RD-CON-2** | If `session_id` omitted, resolve with `WTSGetActiveConsoleSessionId` (or equivalent) — **do not assume SID 1** |
| **C-RD-CON-3** | **Never** bind username on this path (cloud strips it). Username forces Default desktop stick |
| **C-RD-CON-4** | Capture named desktop `Winlogon` first; `OpenInputDesktop` alone is insufficient when Default is active |
| **C-RD-CON-5** | Persistent `capture_method=gdi+black` / all-black frames while claiming `desktop=Winlogon` = **FAIL** (see P0-A) |
| **C-RD-CON-6** | After credentials accepted, switch capture to Default for that session without requiring a second Start |
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
| **C-RD-VIEW-5** | Lock/logon row Start uses C-WL / C-RD-CON start wire (`prefer=winlogon`, `pre_logon`, `desktop=Winlogon`, **no username**) |
| **C-RD-VIEW-6** | On `black_frame` / `winlogon_capture_black` / `CAPTURE_NO_DESKTOP`: explicit degraded banner — never a silent empty player |
| **C-RD-VIEW-7** | Prefer WebRTC when advertised; ICE fail → JPEG-WS ≤2s on the **same** surface |
| **C-RD-VIEW-8** | Show `stream_progress` (`running` → `capturing` → `ws`/`webrtc` → `live`) |
| **C-RD-VIEW-9** | Min client gate: warn if agent &lt; **4.9.26**; recommend ≥ **4.9.49** for console parity acceptance |

---

## Non-goals

- Hypervisor / iLO / Proxmox VGA passthrough (out of agent scope)
- Capturing Session 0 service desktop as a substitute for Winlogon
- Auto-typing credentials from cloud (operator types on stream)
- Embedding noVNC / opening TCP :5900 on agents (optional future transport only)

---

## Cloud (shipped ≥ 1.4.43; viewer cursor ≥ 1.4.47)

- Preserve `pre_logon` / `can_capture` through `normalize_sessions`
- Logon Start: `prefer=winlogon` + `pre_logon` + `desktop=Winlogon`, no username; no hard-coded SID `1`
- UI: Winlogon banner + black-frame honesty (`showWinlogonHint`)
- CAD on Winlogon path omits username
- **1.4.47:** software cursor overlay + C-RD-VIEW-* acceptance on remote console page

---

## Acceptance

### Client ≥ 4.9.49 (or train closing C-RD-CON-*)

- [ ] Empty host (no user logged on): Logon ekranı → non-black logon UI ≤ 10s
- [ ] User Active + lock: sibling Logon row → lock UI, not that user's Default desktop
- [ ] Type password on stream → unlock/logon succeeds → Default desktop appears
- [ ] CAD while on Winlogon → SAS UI (Task Manager / password change options as applicable)
- [ ] Omit `session_id` → still attaches correct console (not wrong RDP session)
- [ ] No sustained `gdi+black` while `desktop=Winlogon` in hello/meta

### Cloud / dashboard

- [x] `pre_logon` preserved in session normalize
- [x] No `session_id=1` hardcode
- [x] Winlogon hint / black-frame UX wired
- [x] **C-RD-VIEW-1** software cursor visible and tracks mouse on Logon ekranı
- [x] **C-RD-VIEW-4..9** CAD / start wire / progress / WebRTC→JPEG≤2s / degraded banner / version warn

---

## Min client

**≥ 4.9.49** recommended for IR console parity.  
Wire floor remains ≥ **4.9.26**; P0 black-frame code ≥ **4.9.45** — this doc raises the **acceptance** bar for "physical console" UX.
