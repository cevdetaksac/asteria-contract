# Remote Desktop — single contract

> **SoT for client + cloud + dashboard.** Contract **≥ 1.4.68** · Agent floor
> **≥ 4.9.95** named topology · **≥ 4.9.97** Logon/empty-host **pixels** (not
> black GDI). Older IDs (C-RD-*, C-WL, C-RD-S0, …) still apply; they live
> **in this file**.
>
> Do not add MUST IDs under stub `agent/remote-*` or `api/05` paths.

**API:** `https://asteria.run`  
**Commands:** `remote_stream_start` / `stop` / `remote_input` / `remote_send_sas`
/ `remote_session_prepare` / `list_sessions`  
**WS:** `wss://…/ws/remote/agent` (agent) · `wss://…/ws/remote/view` (viewer)  
**HTTP (unchanged):** `POST /api/remote/frame`, `frame-json`, `inputs`, `input`,
`session`, `status`, `cad`. No new endpoint for v2. WebRTC signaling is relayed
on the existing agent/view WS.

---

## How to read

| You are | Read |
|---------|------|
| Dashboard / cloud | § Topology, § Viewer, § Smoothness, § CAD cloud |
| Windows agent | § Topology, § Follow, § Session-0, § P0 honesty, § CAD client, § Wire |
| Lab | § Acceptance |

Conflict: **this file wins**.

---

## Topology (C-RD-TOPO) — P0 · ≥4.9.95

Never one Start shape for every Connect. Lab **4.9.94** follow-skip is **not**
acceptance. Lab **4.9.93** FAIL: `prefer=winlogon` on a live Default console →
Winlogon helper + `SESSION0_HELPER_SPAWN_FAILED` + jpeg=0B.

| ID | Rule |
|----|------|
| **C-RD-TOPO-1** | Default Connect / “Logon · varsayılan”: `topology=follow`. **Do not send** `prefer`, `pre_logon`, `desktop`, `session_id`, `username`. **Logged-on console** → DXGI/NVENC `WinSta0\Default`. **Empty host / only LogonUI** → interactive Winlogon helper with **real LogonUI pixels** (same quality as TOPO-2). Never Session-0 BitBlt. Never `gdi+black` / solid-black JPEG claimed as live. |
| **C-RD-TOPO-2** | Logon / Lock **row** (empty host, lock, SAS): `topology=winlogon` + `prefer=winlogon` + `pre_logon=true` + `desktop=Winlogon`. Omit SID and username. Helper is legitimate. |
| **C-RD-TOPO-3** | User shortcut: `session_id` + `username`. Do **not** auto-select the first Active SID. |
| **C-RD-TOPO-4** | After Enter: **same `stream_id`**, no second Start, no “pick administrator”. `WTSGetActiveConsoleSessionId` → `WinSta0\Default`. Winlogon spawn while Default is live must not be a terminal FAIL. |
| **C-RD-TOPO-5** | Min agent **≥4.9.95** topology names. **≥4.9.97** for C-RD-PIX (Logon pixels). Warn if &lt;4.9.26. |

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

---

## Follow after logon (C-RD-FOLLOW)

Same `stream_id`. Dashboard must not ask the operator to pick the user again.

| ID | Rule |
|----|------|
| **C-RD-FOLLOW-1** | After logon/unlock, capture **and** input follow console SID onto `WinSta0\Default`. |
| **C-RD-FOLLOW-2** | Tear down Winlogon helper once LogonUI/SAS is gone and Default is live. |
| **C-RD-FOLLOW-3** | Handle console connect / logon / unlock / secure→Default **before** the next frame. |
| **C-RD-FOLLOW-4** | Do not freeze last Logon JPEG &gt;2s. `phase=switching` then `live`. Growing `age_sec` + `agent_ws_no_frames` after logon = FAIL. |
| **C-RD-FOLLOW-5** | After Default: DXGI / NVENC. Winlogon JPEG ~4 fps is not the post-logon path. |
| **C-RD-FOLLOW-6** | Omit `session_id` = follow (C-RD-TOPO-1). Do not bind first Active SID. |
| **C-RD-FOLLOW-7** | Input targets the **same** desktop as capture. Dual-write Winlogon+Default after logon is forbidden (4× keys). |
| **C-RD-FOLLOW-8** | Live `t:meta` updates `desktop`, `session_id`, `username`, `capture_method` (not start snapshot). |

---

## Console attach (C-RD-CON) — path B

| ID | Rule |
|----|------|
| **C-RD-CON-1** | Honor winlogon prefer/desktop/pre_logon on the **lock row** even if another user is Active |
| **C-RD-CON-2** | Omit SID → `WTSGetActiveConsoleSessionId`. Never invent SID 1. Console 0 → `NO_CONSOLE_SESSION` |
| **C-RD-CON-3** | Never bind username on path B |
| **C-RD-CON-4** | Named `Winlogon` attach before `OpenInputDesktop` Default |
| **C-RD-CON-5** | `gdi+black` / all-black / `persistent-user-helper` GDI with no chrome **while claiming Winlogon** = FAIL (byte-size ≥1500 does **not** make it healthy) |
| **C-RD-CON-6** | After credentials: follow Default without a second Start |
| **C-RD-CON-7** | CAD targets the same console as the stream |
| **C-RD-CON-8** | `list_sessions` always emits Logon/Lock sibling `pre_logon:true` |
| **C-RD-CON-9** | Honest `stream_progress` + `desktop` / `capture_method` / `black_frame` |

---

## Viewer (C-RD-VIEW) — cloud dashboard

Hardware cursor is usually **not** in the bitstream. Draw a local software cursor.

| ID | Rule |
|----|------|
| **C-RD-VIEW-1** | Software cursor over `<video>` / JPEG `<img>` |
| **C-RD-VIEW-2** | Pointer `x,y ∈ [0,1]` vs `meta.native_width/height` + `origin_x/y` (negative origins OK) |
| **C-RD-VIEW-3** | Coalesce only `move`; flush button/wheel/key immediately |
| **C-RD-VIEW-4** | CAD = `remote_send_sas` / `POST /api/remote/cad` only |
| **C-RD-VIEW-5** | Default Connect = TOPO-1; Lock row = TOPO-2 |
| **C-RD-VIEW-6** | `black_frame` / `winlogon_capture_black` / `CAPTURE_NO_DESKTOP` → degraded banner |
| **C-RD-VIEW-7** | WebRTC when advertised; ICE fail → JPEG-WS ≤2s on the **same** surface |
| **C-RD-VIEW-8** | Show `stream_progress` (`running` → `capturing` → `ws`/`webrtc` → `live`) |
| **C-RD-VIEW-9** | Warn agent &lt;4.9.26; recommend ≥4.9.95 |
| **C-RD-VIEW-10** | Do not auto-select first Active user. Frozen frames drop Live badge |

**C-WL (shipped):** show sibling pre_logon row; Winlogon hint; refresh sessions after logon.

---

## Session-0 capture (C-RD-S0 / CL-RD-S0) — ≥4.9.84 on path B

Capture MUST run **in the interactive session** (`CreateProcessAsUser` +
`lpDesktop=winsta0\Winlogon`). Not Session-0 BitBlt.

| ID | Rule |
|----|------|
| **CL-RD-S0-1** | Path B omits `session_id` |
| **CL-RD-S0-2** | Path B never sends `username` |
| **CL-RD-S0-3** | Path B sends winlogon prefer/desktop/pre_logon (not used on TOPO-1) |
| **CL-RD-S0-4** | CAD on Logon: no username / no forced SID |
| **CL-RD-S0-5** | Surface `winlogon_capture_black` / `SESSION0_HELPER_SPAWN_FAILED` — never a silent black player |
| **C-RD-S0-1** | Absent SID → `WTSGetActiveConsoleSessionId`; do not invent `1` |
| **C-RD-S0-3** | Helper in interactive session, not Session 0 |
| **C-RD-S0-5** | jpeg≈0B → `SESSION0_HELPER_SPAWN_FAILED` (`streaming:false`) **unless** TOPO-1 live Default skip/fallback |

---

## Honesty P0 (black + ICE)

| ID | Rule |
|----|------|
| **C-RD-P0-WL-2** | Never healthy media on black-fill only — `black_frame` or fail |
| **C-RD-P0-WL-4** | ≥2s unbroken black → retry once → `winlogon_capture_black` |
| **C-RD-P0-ICE-1** | `connection_state=connected` only after ICE **and** DTLS (or agreed media) |
| **C-RD-P0-ICE-3** | JPEG suppress only while media verified; ICE fail → JPEG-WS ≤2s |
| **C-RD-P0-ICE-5** | ICE failed/closed clears connected UI |

Honesty: `streaming:true` is never faked. No desktop → `NO_INTERACTIVE_SESSION`.
0×0 → `CAPTURE_NO_DESKTOP`. Frames &lt;1500 B rejected. **Solid-black JPEG ≥1500 B
is still black** — see C-RD-PIX.

---

## Pixels / chrome (C-RD-PIX) — P0 · ≥4.9.97

Lab 2026-08-15 Derin-Web (dashboard Default Connect, `topology=follow`): cloud
queued start OK; viewer stayed on **“Görüntü tam değil”**. Agent meta:
`desktop=Winlogon`, `capture_method=persistent-user-helper`, `gdi+black` /
`black_frame`, JPEG ~1024×768 (bytes &gt;1500) **solid black**, WebRTC advertised
(`nvenc`) while picture dead. **This is a client capture FAIL**, not a cloud
viewer bug. Cloud already surfaces VIEW-6 / P0-WL-2.

| ID | Rule |
|----|------|
| **C-RD-PIX-1** | A frame is **healthy** only if it is not a black/flat fill. Tests (any one sufficient, all preferred): `black_frame=false`; `frame_variance` above lab floor; `bright_ratio` shows chrome; `logonui_hwnd_count` ≥ 1 on lock/logon; DXGI of a real Default wallpaper/shell after logon. JPEG size **alone is not** health. |
| **C-RD-PIX-2** | `desktop=Winlogon` (or `capture_method` containing `winlogon` / `gdi`) **MUST** show LogonUI/SAS/credential chrome within **3s** of `capturing`. Else: `black_frame:true`, `phase=degraded` or `failed`, `error=winlogon_capture_black` (or `SESSION0_HELPER_SPAWN_FAILED` if jpeg≈0B). **Never** `streaming:true` + Live. |
| **C-RD-PIX-3** | **Empty host + TOPO-1:** use the **interactive-session helper** (`CreateProcessAsUser` + `lpDesktop=winsta0\Winlogon`) that C-RD-S0 already requires on path B. Do **not** BitBlt the service desktop. Do **not** stop at `persistent-user-helper` GDI black. 4.9.94 “follow-skip” that skips helper **and** yields black = **not acceptance**. |
| **C-RD-PIX-4** | **Logged-on console + TOPO-1:** DXGI/NVENC `WinSta0\Default` only. Spawning Winlogon helper while Default is interactive = FAIL (same as 4.9.93 `SESSION0_HELPER_SPAWN_FAILED` story). |
| **C-RD-PIX-5** | `t:meta.capture_method` MUST be an honest tag, e.g. `dxgi+nvenc`, `persistent-winlogon-helper:raw`, `gdi+black`. `gdi+black` is **never** a success method. |
| **C-RD-PIX-6** | Do **not** offer WebRTC as healthy (`connection_state=connected` / suppress JPEG) until **one** healthy frame (PIX-1) on this `stream_id`. Black + `nvenc` / ICE “connected” = FAIL (P0-ICE). JPEG-WS of chrome is acceptable while ICE runs. |
| **C-RD-PIX-7** | Live `t:meta` every ≤5 frames: `desktop`, `capture_method`, `black_frame`, `frame_variance`, `bright_ratio`, `logonui_hwnd_count`, `session_id`, `username`. Start-command snapshot is not enough. |

**Allowed vs FAIL (lab cheat-sheet)**

| What you see in meta / JPEG | Verdict |
|-----------------------------|---------|
| `dxgi` / `nvenc` + wallpaper or desktop chrome, `black_frame=false` | PASS (TOPO-1 logged-on) |
| `persistent-winlogon-helper:raw` + LogonUI/password box pixels | PASS (empty host or lock row) |
| `gdi+black`, solid 1024×768 black, `persistent-user-helper` + Winlogon | **FAIL** PIX-2/3 |
| jpeg 0 B + `SESSION0_HELPER_SPAWN_FAILED` on TOPO-1 **logged-on** | **FAIL** TOPO-1 |
| jpeg 0 B on **empty** host | **FAIL** S0 unless helper retry then PIX-2 |

---

## CAD + Winlogon input

| ID | Rule |
|----|------|
| **CL-RD-CAD-1** | CAD = `POST /api/remote/cad` → `remote_send_sas` only |
| **CL-RD-CAD-3** | No synthetic `key=ctrl+alt+delete` |
| **C-RD-CAD-3** | `SoftwareSASGeneration` ∈ {1,3}; VOID-only is not success |
| **C-RD-CAD-4** | Unchanged SAS UI → `SAS_NO_EFFECT` |
| **C-RD-IN-WL-1** | Winlogon input → same helper session as capture |
| **C-RD-IN-WL-3** | Meta: `inputs_applied` / `last_input_event` (live, not start snapshot) |

---

## Transport + smoothness

Priority: **WebRTC** (if advertised) → **JPEG-WS** (healthy default) → JPEG-HTTP.
Healthy stream does **not** also POST every frame over HTTP.

- `protocol: 2` = app envelope (`hello`, `meta`, `input`, `input_ack`)
- `protocol: 1` = WebRTC signaling (`webrtc_offer` / answer / reject / ice)
- Non-trickle ICE only
- Cloud offers WebRTC only if `capabilities.webrtc.available` and `"webrtc"∈transports`

| ID | Cloud / viewer |
|----|----------------|
| **C-RD-1** | Offer WebRTC only when advertised |
| **C-RD-2** | While WebRTC connected, do not also paint JPEG |
| **C-RD-4** | Move coalesce OK; key/button/wheel flush immediately |
| **C-RD-5** | ICE reject → JPEG fallback UI at once |
| **C-RD-7** | Restart: drop old peer; new `stream_id` |

---

## Wire (hello / meta)

First text frame on agent WS:

```json
{
  "t": "hello",
  "role": "agent",
  "protocol": 2,
  "stream_id": "…",
  "capabilities": {
    "input_protocols": [1, 2],
    "transports": ["webrtc", "jpeg-ws", "jpeg-http"],
    "fallback": "jpeg-ws",
    "codecs": ["jpeg", "h264"],
    "webrtc": { "available": true, "signaling": 1, "ice": "non-trickle" }
  }
}
```

`t:meta` before binary JPEG (and ≥ every 5th frame): geometry, `session_id`,
`desktop`, `capture_method`, `black_frame`, `frame_variance`, `bright_ratio`,
`logonui_hwnd_count`, `inputs_applied`, adaptive fps/quality.
After follow, meta **must** change (FOLLOW-8). **C-RD-PIX-7.**

`t:stream_progress` (aliases `remote_progress` / `progress`): `phase` =
`running` | `capturing` | `ws` | `webrtc` | `switching` | `live` | `degraded` | `failed`.

Input v2: piggyback on frame ACK; move may coalesce; `mousedown`/`key*` never dropped
behind move flood.

---

## Acceptance (lab) — **client** ticks [`CLIENT_CHECKLIST.md`](./CLIENT_CHECKLIST.md)

Cloud dashboard P0 for RD is done (Connect not auto, follow Start without SID,
honesty banner). Remaining boxes are **agent + lab host**.

### Host prep (do not skip)

1. Note agent version on Derin-Web (or lab). **&lt;4.9.95** cannot close TOPO. **&lt;4.9.97** cannot close PIX.
2. Two physical states, **two separate runs** (reboot or logoff between if needed):
   - **Empty / lock:** no one interactively logged on at console (Logon or Win+L).
   - **Logged-on:** `administrator` (or lab user) **Active** on console, Default desktop visible on the physical screen.
3. Dashboard: `https://asteria.run/dashboard/remote?token=…` — wait idle, **do not** rely on auto-start. Target = **Logon · varsayılan**. Press **Bağlan** only.

### Run A — empty / lock + Default Connect (TOPO-1 + PIX-3)

- [ ] Command params: `topology=follow` only (no prefer/SID).
- [ ] ≤3s: LogonUI or lock chrome **visible** in viewer (password box / legal text / SAS), not a black rectangle.
- [ ] Meta: `black_frame=false`. `capture_method` is helper-raw or equivalent, **not** `gdi+black`.
- [ ] Viewer title is **not** stuck on “Görüntü tam değil” / `Winlogon bağlı — görüntü siyah`.
- [ ] WebRTC may connect **after** chrome exists; black+nvenc is FAIL.

### Run B — empty / lock + Lock row (TOPO-2)

- [ ] Select **Logon / Lock**, Bağlan. Pixels = LogonUI, not a user’s wallpaper.
- [ ] CAD (`remote_send_sas`) advances SAS UI on **this** stream.

### Run C — logged-on console + Default Connect (TOPO-1 + PIX-4)

- [ ] DXGI/NVENC of **that** Default desktop (icons/wallpaper match physical).
- [ ] **No** `SESSION0_HELPER_SPAWN_FAILED`. **No** Winlogon helper spawn.
- [ ] jpeg&gt;0, `black_frame=false`.

### Run D — follow after Enter (FOLLOW)

- [ ] From Run A: type password on the stream, Enter.
- [ ] Same `stream_id`. ≤2s freeze, `phase=switching` then `live` Default shell.
- [ ] Input after logon hits Default only (no 4× keys).

### Run E — ICE honesty

- [ ] Force ICE fail (or wait reject): JPEG-WS chrome ≤2s, no zombie connected.

Historical ticks (do not regress):

- [x] Session-0 empty host Logon pixels path **existed** on **4.9.84** (re-verify PIX on current build — 2026-08-15 lab **regressed** to gdi+black)
- [x] CAD meta honesty (**4.9.86**)
- [x] Cloud Start shapes / CAD / Live honesty ([`../cloud/CLOUD_CHECKLIST.md`](../cloud/CLOUD_CHECKLIST.md))

---

## Out of scope

Hypervisor / iLO / noVNC; auto-typing credentials; Session-0 service desktop as
a fake Winlogon; publishing extra installer names (that is self-update).
