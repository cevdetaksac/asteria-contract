# Remote Desktop HW path (C-RD-HW) — Phase 4

> Contract **≥ 1.4.94**. Extends [`remote-desktop.md`](./remote-desktop.md); topology/follow/CAD IDs there
> still apply. Conflict on `viewer_backend=webrtc_hw`: **this file wins**.
> No RDP/IronRDP, no AGPL/GPL code anywhere in the sidecar or agent.

## 1. Components

| Part | Rule |
|------|------|
| **C-RD-HW-1** `asteria-encode.exe` | Rust sidecar shipped next to the agent (`{app}\bin\asteria-encode.exe`). Permissive deps only (MIT/Apache/BSD/ISC/Zlib). `THIRD_PARTY_LICENSES.md` lists every crate + license; build fails if `cargo deny` finds a copyleft license. |
| **C-RD-HW-2** Run identity | Launched by the agent's SYSTEM service. **Desktop broker** instance runs as SYSTEM in the target session (token duplicated from the service, `SetTokenInformation(TokenSessionId)`, `CreateProcessAsUser` into `winsta0\Winlogon` or `winsta0\Default`). Never a new logon session. |
| **C-RD-HW-3** Capture | DXGI Desktop Duplication (dirty + move rects, cursor shape/position via `GetFramePointerShape`). On `DXGI_ERROR_ACCESS_LOST` / desktop switch: `OpenInputDesktop` + `SetThreadDesktop`, re-create duplication, emit keyframe. GDI BitBlt fallback only for Winlogon if DXGI is refused; flagged `capture=gdi` in stats. |
| **C-RD-HW-4** Encode | Media Foundation H.264 MFT. Order: hardware MFT (NVENC/QSV/AMF via `MFTEnumEx(MFT_ENUM_FLAG_HARDWARE)`) → MS software MFT → OpenH264 (Cisco binary, downloaded at runtime by the user's machine per Cisco's terms; never bundled) → fail ⇒ JPEG fallback. Settings: Baseline/ConstrainedBaseline (or Main w/o B), `CODECAPI_AVLowLatencyMode=1`, B-frames=0, GOP=infinite, IDR on PLI/FIR or desktop switch, CBR/VBR with bitrate set by GCC. **VFR**: no frame when no dirty rects and no cursor-image change (cursor moves go on the data channel). |
| **C-RD-HW-5** Transport | WebRTC (webrtc-rs, MIT/Apache) — 1 video track H.264 packetization-mode=1, trickle ICE, transport-cc + GCC bandwidth estimate drives encoder bitrate (min 300 kbps, start 3 Mbps, max 20 Mbps). ICE servers = cloud-issued short-lived coturn creds (same as today). DTLS-SRTP mandatory. |

## 2. Negotiation (C-RD-NEG)

`remote_stream_start.params` gain:

```json
{
  "viewer_backend": "auto|webrtc_hw|jpeg",
  "session_target": { "kind": "console|session", "session_id": 3 },
  "notify_user": false,
  "monitor": "all|0|1|…"
}
```

| ID | Rule |
|----|------|
| **C-RD-NEG-1** | Default `viewer_backend=auto`: agent tries `webrtc_hw`; on sidecar start failure, encoder unavailable, or no ICE `connected` within **8 s** ⇒ falls back to existing JPEG/aiortc path on the **same `stream_id`** and sends `meta {backend:"jpeg", fallback_reason}`. |
| **C-RD-NEG-2** | Agent `hello` advertises `caps.rd_backends:["jpeg","webrtc_hw"]`, `caps.encoders:["nvenc"|"qsv"|"amf"|"mf_sw"|"openh264"]`. Cloud never asks `webrtc_hw` from an agent without the cap. |
| **C-RD-NEG-3** | Signaling stays on existing `/ws/remote/agent` + `/ws/remote/view` messages `rtc_offer`/`rtc_answer`/`rtc_ice` with `backend:"webrtc_hw"`. Offer comes from the sidecar (via agent stdio pipe). No new HTTP endpoint. |
| **C-RD-NEG-4** | Feature flag `rd_hw_enabled` (cloud per-tenant, default **off** in 4.9.128, on after lab pass). Off ⇒ `auto` behaves as `jpeg`. |
| **C-RD-NEG-SID** | Agent MUST use cloud-provided `stream_id` from start params (do not mint a new UUID). Viewer rejects HW offers whose stream_id does not match the session. |
| **C-RD-NEG-ICE** | On `remote_stream_start`, cloud includes `ice_servers` (TURN 3478/5349 + TURNS 443 + STUN) in command params so the HW sidecar can gather relay candidates (phone 4G/CGNAT). Viewer still receives `webrtc_ice_config`. |

## 3. Console / no-user (C-RD-HW-CON) — HARD

| ID | Rule |
|----|------|
| **C-RD-HW-CON-1** | `session_target.kind=console` works with **no user logged on**: capture of LogonUI on `Winlogon` desktop of the active console session (`WTSGetActiveConsoleSessionId`). |
| **C-RD-HW-CON-2** | Ctrl+Alt+Del: data-channel `{t:"sas"}` ⇒ agent service calls `SendSAS(FALSE)` (requires `SoftwareSASGeneration` policy = 1 or 3; agent sets it at install). Operator then types the password and logs on interactively. |
| **C-RD-HW-CON-3** | Follow: logon, lock (Win+L), unlock, logoff, UAC secure desktop ⇒ the **same** WebRTC connection continues; sidecar switches desktop / respawns in the new console session if the console session id changes, ICE survives (sidecar process for the network side is long-lived; only the per-session capture helper is respawned and pipes frames over shared memory). |
| **C-RD-HW-CON-4** | Process split: `asteria-encode.exe --role net` (SYSTEM, session 0: WebRTC+encode) ↔ `asteria-encode.exe --role capture --session N` (SYSTEM token in session N: DXGI capture + SendInput). IPC = named pipe (control/input) + shared-memory ring of D3D11 shared textures/NV12 frames. Pipe ACL: SYSTEM only. |

## 4. Session picker (C-RD-HW-SES) — HARD

| ID | Rule |
|----|------|
| **C-RD-HW-SES-1** | Viewer shows a **session picker before connect**, filled from `list_sessions` (existing command): rows = Console (always first, label "Console — <user or 'Logon screen'>") + every WTS session `Active` or `Disconnected` (RDP), with user, state, idle, client name. |
| **C-RD-HW-SES-2** | `session_target.kind=session, session_id=N` attaches **live** to that existing session: capture helper spawned in session N with SYSTEM token (`WTSQueryUserToken` used only to resolve the user/desktop; the helper runs as SYSTEM so it can also see Winlogon/UAC of that session). **Never** creates a new session or logs anyone off. |
| **C-RD-HW-SES-3** | Disconnected session: picture is shown if the session has a renderable desktop; otherwise `meta {state:"disconnected_no_display"}` — no reconnect to console without explicit operator action (`tscon` is out of scope). |
| **C-RD-HW-SES-4** | Switching session = `remote_stream_stop` + new start with new target (new `stream_id`). |

## 5. Notify user (C-RD-HW-NOTIFY)

| ID | Rule |
|----|------|
| **C-RD-HW-NOTIFY-1** | Per-connection checkbox in the connect dialog "Notify user when I connect"; **default unchecked (silent)**. Sent as `notify_user:boolean`. |
| **C-RD-HW-NOTIFY-2** | `true` ⇒ agent shows a non-blocking toast/banner in the target session ("<operator> is viewing this screen") for the duration; `false` ⇒ nothing shown. Tenant policy may later force `true`; cloud then overrides the checkbox (`policy.force_notify`). |
| **C-RD-HW-AUDIT-1** | **Every** connection is audit-logged regardless of notify: operator user id, agent/client id, `session_target` (kind, session id, username), backend, notify flag, `start_at`, `end_at`, end reason. Cloud table `remote_audit` (append-only); agent also writes a local event-log entry. |

## 6. Input data channels (C-RD-HW-IN)

Two channels from the viewer: `input-u` (ordered:false, maxRetransmits:0) and `input-r` (ordered, reliable). JSON, short keys.

| Channel | Message | Notes |
|---------|---------|-------|
| input-u | `{t:"mm",x,y,m}` | absolute coords in source pixels of monitor `m`; viewer coalesces to ≤1 per rAF |
| input-u | `{t:"wh",dx,dy}` | wheel, coalesced |
| input-r | `{t:"mb",b:0..4,d:1|0,x,y}` | button down/up, carries position |
| input-r | `{t:"k",sc,e:0|1,d:1|0}` | scan code (`KeyboardEvent.code`→set-1 scan code), `e`=extended, `d`=down |
| input-r | `{t:"txt",s}` | unicode text (IME / paste-as-typing) via `KEYEVENTF_UNICODE` |
| input-r | `{t:"sas"}` | Ctrl+Alt+Del ⇒ SendSAS |
| input-r | `{t:"rel"}` | release all keys/buttons (blur) |
| input-r | `{t:"clip",s}` both ways | text clipboard sync, max 1 MiB; host→viewer sent on clipboard change |
| input-r | `{t:"mon",m}` | switch monitor |
| input-r | `{t:"kf"}` | request keyframe |
| host→viewer (input-r) | `{t:"cur",id,png?,hx,hy}` / `{t:"cp",x,y,v}` | cursor shape (cached by id) / position+visible. Cursor is drawn locally in the browser; it is **not** composited into video. |
| host→viewer | `{t:"mons",list:[{i,x,y,w,h,primary}]}` | monitor list for picker |

Input is injected by the capture helper in the target session (`SendInput`; on Winlogon desktop after `SetThreadDesktop`). Unknown `t` ignored.

## 7. Supervision (C-RD-HW-SUP)

| ID | Rule |
|----|------|
| **C-RD-HW-SUP-1** | Agent starts sidecar on demand, stdio JSON-lines control (`hello`, `offer`, `ice`, `stats`, `bye`, `error`). Health: `stats` every 1 s; 5 s silence ⇒ kill + restart once; second failure in 60 s ⇒ JPEG fallback for this stream. |
| **C-RD-HW-SUP-2** | `--selftest` exits 0 after enumerating encoders/outputs (used by build smoke + agent hello caps). |
| **C-RD-HW-SUP-3** | Stats forwarded to viewer `meta`: fps, encode_ms p50/p95, kbps, target_kbps, encoder, capture (dxgi/gdi), desktop (Default/Winlogon), session_id. |

## 8. Viewer (C-RD-HW-VIEW)

- Session picker + monitor picker + "Notify user" (default off) in the connect dialog.
- `<video autoplay muted playsinline>` for `webrtc_hw`, `playoutDelayHint=0` / `jitterBufferTarget=0` where supported; canvas JPEG path kept as fallback.
- `getStats()` panel: fps, decode, jitter buffer, RTT, kbps, packet loss, codec, candidate pair (host/srflx/relay).
- Keep Phase 3 controls (Fit/1:1, Win, CAD, Alt+Tab, stats); bidirectional text clipboard.

## 9. Acceptance (lab)

| Run | Pass |
|-----|------|
| HW-A | No user logged on ⇒ Console ⇒ LogonUI visible, CAD works, interactive logon, stream continues on Desktop without reconnect. |
| HW-B | Win+L / unlock / UAC prompt followed without reconnect. |
| HW-C | Session picker attaches to an existing RDP session live; no new session in `qwinsta`. |
| HW-D | Smoothness 1080p: ≥ 55 fps on motion, encode p95 ≤ 8 ms on HW MFT, glass-to-glass ≤ 80 ms LAN; idle ⇒ ~0 kbps video. |
| HW-E | Kill sidecar mid-session ⇒ JPEG fallback within 10 s on same stream_id. |
| HW-F | Audit row written with start/end for notify on and off. |

## Rollout

4.9.128: sidecar + agent supervision behind `rd_hw_enabled` (default off). 4.9.129+: default on after HW-A..F pass on DESKTOP-F5SCL3G.
