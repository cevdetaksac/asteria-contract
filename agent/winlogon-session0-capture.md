# Winlogon / Logon UI capture from Session 0 (P0)

> **Contract VERSION:** **1.4.50**  
> Status: **Normative (client P0)**  
> Min client to close lab: **≥ 4.9.84** (or next RD patch that satisfies C-RD-S0-*)  
> Related: [`remote-desktop-p0.md`](./remote-desktop-p0.md) ·
> [`../cloud/remote-console-parity.md`](../cloud/remote-console-parity.md) ·
> [`../cloud/REMOTE_DESKTOP_WINLOGON.md`](../cloud/REMOTE_DESKTOP_WINLOGON.md) ·
> [`../api/05-remote-desktop.md`](../api/05-remote-desktop.md)

## Lab evidence (2026-08-07 — Derin-Web / client **4.9.83**)

Cloud **Logon ekranı → Bağlan** now sends (and MUST send):

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

Observed agent result (≈60ms fail):

| Field | Value |
|-------|--------|
| `error` | `winlogon_capture_black` |
| `capture_method` | `none` |
| `streaming` | `false` |
| message | helper `session=1` `winsta0\Winlogon`; **user-helper jpeg=0B, 0.0s**; **“Agent is Session 0 — capture requires WTSQueryUserToken/CreateProcessAsUser into the selected RDP/console session.”** |

Cloud dashboard shows the honest black/degraded banner (C-RD-VIEW-6).  
**Pixels cannot be fixed by more cloud params** — Session-0 helper spawn / Winlogon desktop capture must work on the agent.

---

## Cloud wire (shipped ≥ 1.4.50)

| ID | Rule |
|----|------|
| **CL-RD-S0-1** | Logon/lock Start **omits** `session_id` (C-RD-CON-2). Agent resolves console. |
| **CL-RD-S0-2** | Logon/lock Start **never** sends `username`. |
| **CL-RD-S0-3** | Always send `prefer=winlogon` + `pre_logon=true` + `desktop=Winlogon`. |
| **CL-RD-S0-4** | CAD on Logon path: `remote_send_sas` without username / without forced SID. |
| **CL-RD-S0-5** | Surface `winlogon_capture_black` / `gdi+black` / `capture_method=none` — never silent black player. |

---

## Client MUST (C-RD-S0-*)

| ID | Rule |
|----|------|
| **C-RD-S0-1** | When `session_id` is **absent**, resolve with `WTSGetActiveConsoleSessionId` (or equivalent). **Do not invent SID `1`** if console resolve yields another id or fails. |
| **C-RD-S0-2** | When `session_id` is present, use it; still attach **`WinSta0\Winlogon`** named desktop **before** Default / `OpenInputDesktop`. |
| **C-RD-S0-3** | Agent service in **Session 0 MUST NOT** BitBlt/DXGI from Session 0. Capture MUST run **in the target interactive session** via helper: `WTSQueryUserToken` → `CreateProcessAsUser` (or equivalent) with `lpDesktop = "winsta0\\Winlogon"`. |
| **C-RD-S0-4** | Pre-logon / empty username: `WTSQueryUserToken(sid)` often fails (no logged-on user). Agent MUST fall back to an interactive token usable in that session — at minimum: locate `winlogon.exe` (or `LogonUI.exe`) **in the console session**, duplicate its primary token, then `CreateProcessAsUser` helper with Winlogon desktop. Do **not** stop after a single empty `WTSQueryUserToken`. |
| **C-RD-S0-5** | Helper spawn that returns **jpeg=0B in ≈0s** is a **hard spawn/attach failure**, not a slow black GDI. Fail with distinct reason when possible: `SESSION0_HELPER_SPAWN_FAILED` (or keep `winlogon_capture_black` **and** set `capture_method=none`, `streaming:false`, honest `message`). Never claim media connected. |
| **C-RD-S0-6** | After helper starts: wait until first frame **≥ 1500 B** (real UI pixels) **or** ≥2s unbroken black → one DXGI/BitBlt retry → then `winlogon_capture_black` (existing C-RD-P0-WL-4). |
| **C-RD-S0-7** | Success meta MUST include `desktop=Winlogon` (or lock desktop name), non-`none` `capture_method`, `streaming:true`, and non-black frames. |
| **C-RD-S0-8** | After credentials accepted on stream, switch capture to **Default** for that session **without** requiring a second Start (C-RD-CON-6). |
| **C-RD-S0-9** | `remote_send_sas` targets the **same** console session the stream is on; works with Session-0 agent (elevated SendSAS / same helper session). |
| **C-RD-S0-10** | Hello capabilities may advertise `winlogon`/`pre_logon` **only** when Session-0→interactive helper path is implemented; otherwise do not advertise (avoid operator false confidence). |

---

## Forbidden

- Capturing Session 0 service desktop and labeling it Winlogon  
- Hardcoding `session_id = 1` when params omit SID  
- Reporting `success:true` / `streaming:true` with `capture_method=none` or all-black frames  
- Binding `username` on `prefer=winlogon` path (forces Default stick)

---

## Acceptance (lab)

Host: Windows console at **Logon / Lock** UI; agent service **Session 0**; cloud Logon row Connect (no SID in params).

- [ ] Start completes with visible logon/lock UI pixels ≤ **3s** (not black)  
- [ ] Result has `streaming:true`, `capture_method` ≠ `none`, frames ≥1500 B  
- [ ] Omit `session_id` → attaches **active console**, not a random RDP SID  
- [ ] jpeg=0B / 0.0s helper path does not happen on healthy hosts  
- [ ] User Active + sibling Logon row → lock/logon UI (not that user's Default)  
- [ ] Type password on stream → Default desktop without second Start  
- [ ] CAD → SAS UI on same session  

Close-out: bump client floor in [`../FLEET.md`](../FLEET.md) when lab rows pass; keep error code `winlogon_capture_black` for persistent black after honest retries.

---

## Operator handoff (one paragraph for client chat)

> Contract **1.4.50** / `agent/winlogon-session0-capture.md`. Cloud already omits `session_id` on Logon Start. **4.9.83** still fails in ~60ms with helper `session=1`, jpeg=0B, Session-0 CreateProcessAsUser path. Implement C-RD-S0-1…10: resolve console without inventing SID 1; spawn capture helper **into** interactive session on `winsta0\Winlogon`; if `WTSQueryUserToken` fails on pre-logon, duplicate Winlogon/LogonUI token in that session. Ship ≥**4.9.84** (or this train) with lab acceptance checked.
