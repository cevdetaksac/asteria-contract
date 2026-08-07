# Remote Desktop P0 — Winlogon black-screen & WebRTC ICE honesty

> **Contract VERSION:** **1.4.50** (P0 originally **1.4.37**)  
> Status: **Normative (client P0)**  
> Cloud C-WL / smoothness mostly shipped; **client bugs remain open** and
> mislead operators during IR.  
> **Session-0 spawn path (2026-08 lab):** see
> [`winlogon-session0-capture.md`](./winlogon-session0-capture.md) (**C-RD-S0-***,
> client ≥**4.9.84**).  
> Related: [`../cloud/REMOTE_DESKTOP_WINLOGON.md`](../cloud/REMOTE_DESKTOP_WINLOGON.md) ·
> [`../cloud/REMOTE_DESKTOP_SMOOTHNESS.md`](../cloud/REMOTE_DESKTOP_SMOOTHNESS.md) ·
> [`remote-stream-progress.md`](./remote-stream-progress.md) ·
> [`../api/05-remote-desktop.md`](../api/05-remote-desktop.md)

**Priority: P0.** Both classes make the operator believe the host is blank or
“connected” when it is not — wrong IR decisions.

Min client target for close-out: **≥ 4.9.84** for Session-0 Logon pixels
(C-RD-S0); honesty codes from **≥ 4.9.37** / **≥ 4.9.45**.

---

## P0-A — Winlogon / pre_logon black screen (`gdi+black`)

### Symptom
Lock / Logon sibling selected; stream starts with `prefer=winlogon` /
`desktop=Winlogon`; frames are solid black; `capture_method` reports
`gdi+black` (or equivalent). Operator thinks console is dead.

### Client MUST (C-RD-P0-WL-*)

| ID | Rule |
|----|------|
| **C-RD-P0-WL-1** | Named `Winlogon` desktop attach **before** falling back to `OpenInputDesktop` Default |
| **C-RD-P0-WL-2** | Never report healthy media while capture is black-fill only — surface `capture_method` + `black_frame:true` (or fail stream with explicit reason) |
| **C-RD-P0-WL-3** | Honor cloud start params: `prefer=winlogon`, `pre_logon=true`, `desktop=Winlogon`, **no username** on lock row |
| **C-RD-P0-WL-4** | If GDI returns unbroken black for **≥2s** after start → retry DXGI/BitBlt path once, then emit control error `winlogon_capture_black` (do not silently “succeed”) |
| **C-RD-P0-WL-5** | Health/`list_sessions` always includes sibling **Logon / Lock screen** row (`pre_logon:true`) per 1.4.23 |
| **C-RD-P0-WL-6** | Session-0 agents: follow **C-RD-S0-*** ([`winlogon-session0-capture.md`](./winlogon-session0-capture.md)) — helper jpeg=0B / invent SID 1 is FAIL |

### Acceptance
- [x] Client code (≥4.9.45): Winlogon attach, `black_frame`, `winlogon_capture_black`, ICE honesty, JPEG fallback  
- [ ] Lab: Locked console → Start on lock row → visible logon UI pixels (not black) within 3s  
- [ ] Lab: User session row still starts Default (not forced Winlogon)  
- [ ] Lab: Black-fill path never advertises `connection_state=connected` without `degraded`/`black_frame`  
- [x] Lab (1.4.50→1.4.51): cloud omits `session_id` → Session-0 helper captures Winlogon pixels (**4.9.84**)

---

## P0-B — WebRTC ICE / media honesty

### Symptom
Viewer shows connected / media path “up” while ICE stuck in `checking`, or
JPEG suppressed while DTLS never completes — operator waits on a dead pipe.

### Client MUST (C-RD-P0-ICE-*)

| ID | Rule |
|----|------|
| **C-RD-P0-ICE-1** | Do **not** set `media.connection_state=connected` until ICE **and** DTLS (or agreed media path) are actually up |
| **C-RD-P0-ICE-2** | Telemetry must expose honest `ice_state` (`new`/`checking`/`connected`/`failed`/`closed`) on the wire the dashboard already consumes |
| **C-RD-P0-ICE-3** | Strict JPEG suppression **only** while ICE+DTLS media is verified; if ICE fails/times out → JPEG-WS fallback **≤2s** (see smoothness C-RD) |
| **C-RD-P0-ICE-4** | Non-trickle ICE only; reject standalone trickle with `reason=non_trickle_ice` (unchanged) |
| **C-RD-P0-ICE-5** | On ICE `failed`/`disconnected`/`closed`, clear “connected” UI signals and emit explicit reason (no zombie connected) |

### Acceptance
- [x] Client code (≥4.9.45): ICE+DTLS before connected; jpeg_fallback_active; fail clears connected  
- [ ] Lab: block UDP/TURN → UI shows checking→failed (not connected); JPEG fallback ≤2s  
- [ ] Lab: healthy path → ICE connected before first H264 frame claimed  
- [ ] Lab: No “connected” toast while `ice_state=checking` for >15s without timeout UX

---

## Out of scope (this P0)

- New trickle ICE designs  
- Changing cloud C-WL start wire (already shipped)  
- Non-console session helpers unrelated to black/ICE honesty

---

## Tracking

Close both P0 rows in the same client release notes; contract stays **1.4.37**
until acceptance checked — then bump client floor in [`../FLEET.md`](../FLEET.md).
