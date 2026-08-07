# Winlogon / Logon UI capture from Session 0 (P0)

> **Contract VERSION:** **1.4.51** (opened **1.4.50**)  
> Status: **Accepted — client ≥ 4.9.84** (lab Derin-Web 2026-08-07)  
> Min client: **≥ 4.9.84**  
> Related: [`remote-desktop-p0.md`](./remote-desktop-p0.md) ·
> [`../cloud/remote-console-parity.md`](../cloud/remote-console-parity.md) ·
> [`../cloud/REMOTE_DESKTOP_WINLOGON.md`](../cloud/REMOTE_DESKTOP_WINLOGON.md) ·
> [`../api/05-remote-desktop.md`](../api/05-remote-desktop.md)

## Lab evidence

### Fail baseline (2026-08-07 — Derin-Web / **4.9.83**)

Cloud **Logon ekranı → Bağlan** sends (and MUST send):

```json
{
  "command_type": "remote_stream_start",
  "params": {
    "prefer": "winlogon",
    "pre_logon": true,
    "desktop": "Winlogon",
    "preferred_transport": "webrtc",
    "transports": ["webrtc", "websocket"],
    "stream_id": "…",
    "fps": 24, "quality": 40, "max_width": 1280
  }
}
```

**No `session_id`. No `username`.**

| Field | Value |
|-------|--------|
| `error` | `winlogon_capture_black` |
| `capture_method` | `none` |
| `streaming` | `false` |
| message | helper jpeg=0B / Session 0 CreateProcessAsUser miss |

### Pass (2026-08-07 — Derin-Web / **4.9.84**)

| Field | Value |
|-------|--------|
| `success` | `true` |
| `streaming` | `true` |
| `capture_method` | `persistent-winlogon-helper:raw` |
| `winlogon_mode` | `true` |
| `black_frame` | `false` |
| frames | ≥1, `black_frames=0` |
| `execution_time_ms` | ~2600 |
| Viewer | Lock/LogonUI visible (Ctrl+Alt+Delete prompt) via JPEG-WS |

Release: https://github.com/cevdetaksac/asteria-client/releases/tag/v4.9.84

---

## Cloud wire (shipped ≥ 1.4.50)

| ID | Rule |
|----|------|
| **CL-RD-S0-1** | Logon/lock Start **omits** `session_id` (C-RD-CON-2). Agent resolves console. |
| **CL-RD-S0-2** | Logon/lock Start **never** sends `username`. |
| **CL-RD-S0-3** | Always send `prefer=winlogon` + `pre_logon=true` + `desktop=Winlogon`. |
| **CL-RD-S0-4** | CAD on Logon path: `remote_send_sas` without username / without forced SID. |
| **CL-RD-S0-5** | Surface `winlogon_capture_black` / `SESSION0_HELPER_SPAWN_FAILED` / `gdi+black` — never silent black player. |

---

## Client MUST (C-RD-S0-*) — implemented ≥ **4.9.84**

| ID | Rule |
|----|------|
| **C-RD-S0-1** | When `session_id` is **absent**, resolve with `WTSGetActiveConsoleSessionId`. **Do not invent SID `1`**; console `0` → `NO_CONSOLE_SESSION`. |
| **C-RD-S0-2** | When `session_id` is present, use it; attach **`WinSta0\Winlogon`** before Default / `OpenInputDesktop`. |
| **C-RD-S0-3** | Capture MUST run **in the interactive session** via helper (`CreateProcessAsUser` + `lpDesktop = "winsta0\\Winlogon"`). Not Session 0 BitBlt. |
| **C-RD-S0-4** | Token chain: `WTSQueryUserToken` → winlogon/LogonUI process token → SYSTEM+`TokenSessionId`. |
| **C-RD-S0-5** | jpeg≈0B / ~0s → `SESSION0_HELPER_SPAWN_FAILED` (`streaming:false`, `capture_method=none`). |
| **C-RD-S0-6** | First frame ≥1500 B; 2s black → retry → `winlogon_capture_black`. |
| **C-RD-S0-7** | Success: non-`none` `capture_method`, `streaming:true`, non-black frames. |
| **C-RD-S0-8** | After credentials accepted, switch to Default without second Start (C-RD-CON-6). |
| **C-RD-S0-9** | `remote_send_sas` same console session as stream. |
| **C-RD-S0-10** | Advertise `winlogon`/`pre_logon` only when Session-0→interactive helper path exists. |

---

## Forbidden

- Capturing Session 0 service desktop and labeling it Winlogon  
- Hardcoding `session_id = 1` when params omit SID  
- Reporting `success:true` / `streaming:true` with `capture_method=none` or all-black frames  
- Binding `username` on `prefer=winlogon` path

---

## Acceptance (lab)

Host: Windows console at **Logon / Lock** UI; agent service **Session 0**; cloud Logon row Connect (no SID in params).

- [x] Start completes with visible logon/lock UI pixels ≤ **3s** (not black) — **4.9.84 / Derin-Web**  
- [x] Result has `streaming:true`, `capture_method` ≠ `none`, frames ≥1500 B  
- [x] Omit `session_id` → attaches **active console** (no invent SID 1)  
- [x] jpeg=0B / 0.0s helper path does not happen on healthy hosts  
- [ ] User Active + sibling Logon row → lock/logon UI (not that user's Default) — optional follow-up  
- [ ] Type password on stream → Default desktop without second Start — optional follow-up  
- [ ] CAD → SAS UI on same session — optional follow-up  

**P0 pixel path closed.** Remaining rows are console UX polish (C-RD-CON-6/7), not Session-0 spawn.
