# Remote Desktop — single contract

> **SoT for client + cloud + dashboard.** Contract **≥ 1.4.87** · Agent floor
> **≥ 4.9.95** named topology · **≥ 4.9.100** physical-console **pixels**
> · **≥ 4.9.101** video-rate stream (C-RD-SMOOTH) · recommend **≥ 4.9.120**
> (Capture health honesty / no Live flicker; prior **≥4.9.119** dumps /
> **≥4.9.118** Default black recovery).
> **4.9.99–4.9.109** Derin-Web `persistent-user-helper` + `gdi+black` / `dxgi:pending`
> labs are **not** acceptance. Frozen Welcome after password without Default **pixels**
> is **not** acceptance (FOLLOW-4). Older IDs still apply; they live **in this file**.
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
| **C-RD-TOPO-1** | Default Connect / “Logon · varsayılan”: `topology=follow`. **Do not send** `prefer`, `pre_logon`, `desktop`, `session_id`, `username`. Agent follows the **physical input desktop**: locked/empty → Winlogon helper + LogonUI pixels; unlocked Default → DXGI/NVENC. A WTS username still listed is **not** proof Default is live (Win+L). Never Session-0 BitBlt. Never `gdi+black` claimed as live. |
| **C-RD-TOPO-2** | Logon / Lock **row** (empty host, lock, SAS): `topology=winlogon` + `prefer=winlogon` + `pre_logon=true` + `desktop=Winlogon`. Omit SID and username. Helper is legitimate. |
| **C-RD-TOPO-3** | User shortcut: `session_id` + `username`. Do **not** auto-select the first Active SID. |
| **C-RD-TOPO-4** | After Enter **or** unlock: **same `stream_id`**, no second Start, no “pick administrator”. After **lock / logoff**: same `stream_id` back to Winlogon helper. Winlogon spawn while Default is actually live must not be a terminal FAIL. |
| **C-RD-TOPO-5** | Min agent **≥4.9.95** topology names. **≥4.9.100** for C-RD-PIX + lock/logoff follow. **≥4.9.101** for C-RD-SMOOTH. Warn if &lt;4.9.26. Recommend **≥4.9.110**. 4.9.94 follow-skip and **4.9.99–4.9.109** `gdi+black` / `persistent-user-helper` / `dxgi:pending` lab are **not** acceptance. |

**A — Default Connect**

```json
{
  "command_type": "remote_stream_start",
  "params": { "topology": "follow", "stream_id": "…", "fps": 30, "quality": 72, "max_width": 1920 }
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
    "fps": 30, "quality": 72, "max_width": 1920
  }
}
```

---

## Follow after logon (C-RD-FOLLOW)

Same `stream_id`. Dashboard must not ask the operator to pick the user again.

| ID | Rule |
|----|------|
| **C-RD-FOLLOW-1** | After logon/unlock, capture **and** input follow console SID onto `WinSta0\Default`. |
| **C-RD-FOLLOW-2** | Tear down Winlogon helper once LogonUI/SAS is gone and Default is live (or Welcome → shell). |
| **C-RD-FOLLOW-3** | Handle console connect / logon / unlock / secure→Default **before** the next frame. |
| **C-RD-FOLLOW-4** | Do not freeze last Logon JPEG &gt;2s. `phase=switching` then `live`. Growing `age_sec` + `agent_ws_no_frames` after logon = FAIL. Banner “Yayın durdu / konsol takibi” without Default = FAIL. Agent **≥4.9.118**: after switch to `desktop=Default`, recover `gdi+black` / `no_frame` via DXGI retry (explorer optional) then Active Console/RDP helper respawn (`active-rdp-fallback:*`); keep `phase=degraded` until PIX-1 healthy — never paint frozen Welcome as Live. |
| **C-RD-FOLLOW-5** | After Default: DXGI / NVENC. Winlogon JPEG ~4 fps is not the post-logon path. |
| **C-RD-FOLLOW-6** | Omit `session_id` = follow (C-RD-TOPO-1). Do not bind first Active SID. |
| **C-RD-FOLLOW-7** | Input targets the **same** desktop as capture. Dual-write Winlogon+Default after logon is forbidden (4× keys). |
| **C-RD-FOLLOW-8** | Live `t:meta` updates `desktop`, `session_id`, `username`, `capture_method` (not start snapshot). |
| **C-RD-FOLLOW-9** | After **lock / logoff**: same `stream_id`, `phase=switching` then Winlogon helper (`CreateProcessAsUser` + `lpDesktop=winsta0\\Winlogon` + **winlogon.exe / LogonUI token**, not the logged-on user token). LogonUI chrome must return. |
| **C-RD-FOLLOW-10** | Switch to Default when unlock is proven: LogonUI gone + WTS unlocked (explorer optional during “Windows is getting ready”), **or** live input desktop is Default, **or** (agent ≥4.9.117) WTS unlocked + username even while LogonUI.exe briefly lingers on Welcome (`post_logon_welcome`). **FAIL** to treat `list_sessions` username / Active alone as unlock while lock UI is still interactive (`session_locked` not False). Stale `desktop=Winlogon` / lock-row `force_secure` must **not** block post-password follow. |
| **C-RD-FOLLOW-11** | Lock-row Start (`topology=winlogon` / `force_secure`) **must** still follow Default after credentials on the **same** `stream_id` (no Durdur/Bağlan). |

---

## Capture diagnostics (C-RD-DIAG) — ≥4.9.107

Host A (e.g. WIN-6E5…) may show LogonUI while Derin-Web / Ninety-Web stay
`gdi+black` or freeze after password. Operators need **comparable** fields, not
only a red banner.

| ID | Rule |
|----|------|
| **C-RD-DIAG-1** | Agent emits `t:capture_diag` (protocol 2) on Start probe fail, `switching`, post-logon degraded/live, and includes the same object under `t:meta.capture_diag`. |
| **C-RD-DIAG-2** | Fields (min): `desktop`, `capture_method`, `winlogon_mode`, `helper_token`, `helper_fail_phase`, `session_id`, `username`, `black_frame`, `frame_variance`, `bright_ratio`, `logonui_hwnd_count`, `chrome_detected`, `follow_console`, `force_secure`, `frames_sent`. |
| **C-RD-DIAG-3** | Dashboard **Capture health** panel: green/red per field; copy JSON; compare last connect across hosts. Do **not** only show “görüntü siyah”. |
| **C-RD-DIAG-4** | Viewer “konsol takibi client'ta” / frozen Welcome: if `capture_diag.phase` never reaches Default/`live`, treat as client FOLLOW FAIL (pin ≥4.9.107); if agent ≥4.9.107 and still stuck, surface `helper_fail_*` in the panel. |
| **C-RD-DIAG-5** | Agent **≥4.9.112**: when pixels are unhealthy, emit rich `capture_diag` (≥ every 2s + on probe/fail) with `healthy`, `layer`, `faults[]`, `root_cause`, `advice`, `blame` (`client` / `network_or_cloud` / `webrtc_optional`). Derin `LOGONUI_PRESENT_BUT_FLAT` must set `blame=client` + `layer=client_capture`. Also include the same object on `remote_stream_start` result `data.capture_diag` and `t:meta.capture_diag`. Agent **≥4.9.120**: when pixels recover, clear stale `helper_fail_phase=no_frame` and emit `phase=live` so cloud replaces FAIL. |
| **C-RD-DIAG-6** | Agent **≥4.9.119**: on Start terminal fail (`SESSION0_HELPER_NO_FRAME`, `CAPTURE_NO_DESKTOP`, winlogon flat/black), sustained Default/Winlogon empty frames (`default_no_frame*`), follow `FOLLOW_NO_DEFAULT_FRAME`, and Default black recover fail — write a local dump under `%ProgramData%\Asteria\rd_capture_diag\` (JSON + optional JPEG) and include `local_dump_path`, `recovery_steps[]`, `hwnd_classes[]` on `capture_diag`. Cloud should show `local_dump_path` in Capture health JSON. (Earlier ≥4.9.115 covered winlogon flat only.) |
| **C-RD-DIAG-7** | Agent **≥4.9.120** + cloud: if `healthy=true` / `blame=none` / `black_frame=false` and JPEG-WS frames arrive with healthy `frame_variance`, Capture health is **PASS/OK** and Live stays on. Do **not** drop Live for `blame=webrtc_optional`, media WebRTC peer errors under websocket-primary, or stale `helper_fail_phase=no_frame` / `jpeg=0B` while chrome variance is healthy. Do **not** banner “persistent-helper + black — Capture FAIL” when `black_frame=false` and method contains `printwindow-logonui`. |

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
| **C-RD-VIEW-8** | Show `stream_progress` (`running` → `capturing` → `ws`/`webrtc` → `switching` → `live`). Honor `switching` on unlock **and** lock/logoff. |
| **C-RD-VIEW-9** | Warn agent &lt;4.9.26; recommend **≥4.9.107** (post-logon follow + `capture_diag` + PIX). Do not cite “≥4.9.45 P0” / “≥4.9.93” as the current floor on frozen-frame banners. |
| **C-RD-VIEW-10** | Do not auto-select first Active user. Frozen frames drop Live badge |
| **C-RD-VIEW-11** | Default Connect / follow must **not** auto-open a dashboard “Kullanıcıya bağlan” password modal over the player. Operator types on the remote surface (physical console). Optional shortcut is a separate action. |

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

## Pixels / chrome (C-RD-PIX) — P0 · ≥4.9.100 (recommend ≥4.9.107)

Lab **2026-08-17 Derin-Web on 4.9.103**: Default Connect `topology=follow` and
`administrator` SID Start; viewer “Winlogon bağlı — görüntü siyah”; meta
`capture_method=persistent-user-helper`, `gdi+black` / `black_frame=true`,
JPEG ~1024×768, WebRTC failed (`nvenc` · ~8 fps), encode meta ~Q40. Same FAIL
pattern as 4.9.99–4.9.102. **Cloud honesty OK (no Live).** Capture chrome is
still missing — do **not** tick PIX until Runs A–C PASS. Version string alone
is not acceptance.

| ID | Rule |
|----|------|
| **C-RD-PIX-1** | A frame is **healthy** only if it is not a black/flat fill. Tests (any one sufficient, all preferred): `black_frame=false`; `frame_variance` above lab floor; `bright_ratio` shows chrome; `logonui_hwnd_count` ≥ 1 on lock/logon; DXGI of a real Default wallpaper/shell after logon. JPEG size **alone is not** health. Agent **≥4.9.114**: on Winlogon prefer PrintWindow / HWND BitBlt over desktop BitBlt; require `desktop_attached` (helper-mirrored OK). Agent **≥4.9.116**: PrintWindow uses RedrawWindow + child HWND enum + retry; may try DXGI on Winlogon when GDI is flat. |
| **C-RD-PIX-2** | `desktop=Winlogon` (or `capture_method` containing `winlogon` / `gdi`) **MUST** show LogonUI/SAS/credential chrome within **3s** of `capturing`. Else: `black_frame:true` / `flat_frame:true`, `phase=degraded` or `failed`, `error=winlogon_capture_black` / `winlogon_capture_flat` (or `SESSION0_HELPER_SPAWN_FAILED` if jpeg≈0B). **Never** `success:true` + `frames_sent:0` as Live. `gdi+flat` with LogonUI hwnd ≥1 = FAIL (not soft-degraded). Agent **≥4.9.116**: before terminal FAIL, attempt **C-RD-HOST-2** Active RDP Default fallback when an Active user session exists. |
| **C-RD-PIX-3** | **Empty host + TOPO-1:** use the **interactive-session helper** (`CreateProcessAsUser` + `lpDesktop=winsta0\Winlogon` + winlogon token) that C-RD-S0 already requires on path B. Do **not** BitBlt the service desktop. Do **not** stop at `persistent-user-helper` GDI black. 4.9.94 “follow-skip” that skips helper **and** yields black = **not acceptance**. |
| **C-RD-PIX-4** | **Logged-on console + TOPO-1:** DXGI/NVENC `WinSta0\Default` only. Spawning Winlogon helper while Default is interactive = FAIL (same as 4.9.93 `SESSION0_HELPER_SPAWN_FAILED` story). Agent **≥4.9.118**: DXGI retry after `gdi+black` must **not** require `explorer.exe` (Welcome / “Hoş Geldiniz” has no shell yet). |
| **C-RD-PIX-5** | `t:meta.capture_method` MUST be an honest tag, e.g. `dxgi+nvenc`, `persistent-winlogon-helper:raw`, `gdi+black`, `active-rdp-fallback:rdp`, `dxgi-winlogon`. `gdi+black` is **never** a success method. |

---

## Host prep / Server SKU (C-RD-HOST) — ≥4.9.116

Host A (Billur / `WIN-6E5…`) may PrintWindow LogonUI while Derin-Web /
Ninety-Web (Windows Server VMs) stay `gdi+flat` even with hwnd≥1. Classic RDP
still works because it owns TermService pixels — Asteria must mirror console
composition **or** fall back to an Active RDP session.

| ID | Rule |
|----|------|
| **C-RD-HOST-1** | Installer + daemon apply idempotent host prep: Themes/DWM/TermService start, `fDenyTSConnections=0`, clear `DisallowComposition`, DWM `ForceEffectMode`, monitor power timeout 0, `%ProgramData%\Asteria\rd_capture_diag\`, Server `AutoRestartShell=1`. |
| **C-RD-HOST-2** | When console Winlogon start is flat/black after PrintWindow settle **or** post-logon Default paints `gdi+black` / `no_frame` (agent ≥4.9.118) **and** an **Active** user session exists (prefer RDP, else Console), switch/respawn helper on that session’s `Default` and stamp `capture_method` with `active-rdp-fallback:*`. Do not claim Winlogon chrome for that path. Same-SID Console after unlock is allowed (Welcome DXGI miss). |

---

## Pixels / chrome (C-RD-PIX) — continued

| ID | Rule |
|----|------|
| **C-RD-PIX-6** | Do **not** offer WebRTC as healthy (`connection_state=connected` / suppress JPEG) until **one** healthy frame (PIX-1) on this `stream_id`. Black + `nvenc` / ICE “connected” = FAIL (P0-ICE). JPEG-WS of chrome is acceptable while ICE runs. |
| **C-RD-PIX-7** | Live `t:meta` every ≤5 frames: `desktop`, `capture_method`, `black_frame`, `frame_variance`, `bright_ratio`, `logonui_hwnd_count`, `session_id`, `username`. Start-command snapshot is not enough. |

**Allowed vs FAIL (lab cheat-sheet)**

| What you see in meta / JPEG | Verdict |
|-----------------------------|---------|
| `dxgi` / `nvenc` + wallpaper or desktop chrome, `black_frame=false` | PASS (TOPO-1 logged-on) |
| `persistent-winlogon-helper:raw` + LogonUI/password box pixels | PASS (empty host, lock, or after logoff) |
| `gdi+black`, solid 1024×768 black, `persistent-user-helper` + Winlogon | **FAIL** PIX-2/3 / FOLLOW-10 |
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

## Transport + smoothness (C-RD-SMOOTH) — ≥4.9.101 · primary path ≥4.9.110

**Primary continuous video = JPEG-WS** on the agent outbound tunnel
(`wss://…/ws/remote/agent` → viewer). High-fps JPEG frames **are** the live
picture (like watching video), not a “degraded mode”. Hosts and operators
commonly have **≥100 Mbit**; servers often **≥1–10 Gbit** — do not ship a
12 fps / Q40 slideshow.

**WebRTC** is an **opportunistic upgrade** when advertised **and** ICE/DTLS
actually reach `connected`. Cloudflare-proxied origins often block host UDP;
waiting forever on ICE while starving JPEG-WS is FAIL. While ICE negotiates,
the agent encodes JPEG in-helper (`prefer_raw=false`) so the tunnel stays
smooth; raw RGB is only for a connected WebRTC peer.

Priority for paint: **WebRTC connected** → **JPEG-WS** (always running until
then) → JPEG-HTTP. Healthy stream does **not** also POST every frame over HTTP.
Wire field `fallback:"jpeg-ws"` is **legacy naming** — treat it as the default
live path.

- `protocol: 2` = app envelope (`hello`, `meta`, `input`, `input_ack`)
- `protocol: 1` = WebRTC signaling (`webrtc_offer` / answer / reject / ice).
  Agent **≥4.9.101** treats omitted protocol on offer/answer/ice as `1`.
- Non-trickle ICE only
- Cloud offers WebRTC only if `capabilities.webrtc.available` and `"webrtc"∈transports`
- `hello.capabilities.webrtc.needs_turn` + `preferred_ice: turns`
- `hello.capabilities.smoothness` = capture_fps / jpeg_fallback_fps / max_width / target_bitrate_bps

| ID | Rule |
|----|------|
| **C-RD-1** | Offer WebRTC only when advertised |
| **C-RD-2** | While WebRTC connected, do not also paint JPEG |
| **C-RD-4** | Move coalesce OK; key/button/wheel flush immediately on agent WS |
| **C-RD-5** | ICE reject / stall → keep JPEG-WS UI **at video rate** (not 8–12 fps). Do **not** blank the player waiting on ICE. |
| **C-RD-7** | Restart: drop old peer; new `stream_id` |
| **C-RD-SMOOTH-1** | Start knobs: `fps` **≥30** (clamp 30–60), `quality` **≥72** (55–90), `max_width` **1920**. Do **not** send `fps:12` / `quality:40` / `max_width:1280`. Agent ≥4.9.101 lifts those legacy values itself; older agents will slideshow. |
| **C-RD-SMOOTH-2** | Include `ice_servers` on the offer: **TURNS** (TLS) on **443** plus STUN. Cloudflare-proxied origins often block host UDP; TURNS:443 is the WebRTC path that can work. |
| **C-RD-SMOOTH-3** | Target encode **1080p30–60**, ~8–12 Mbps when the viewer reports ≥50 Mbit. Do not cap at 1 Mbps / 30 fps in the peer. |
| **C-RD-SMOOTH-4** | JPEG-WS latest-frame coalescing is **correct** (drop stale). Do not treat coalesced count as congestion that lowers fps. |
| **C-RD-SMOOTH-5** | Viewer: decode/paint as video (`requestAnimationFrame` / `<video>`), not a 12 Hz `<img>` refresh. Prefer WebRTC `<video>` only when ICE is connected; otherwise rAF JPEG-WS is Live. |
| **C-RD-SMOOTH-6** | Recommend agent **≥4.9.110** for JPEG-WS primary + honest method stamp + lock probe. PIX/lock-follow floor remains **≥4.9.100**; SMOOTH wire floor **≥4.9.101**. |
| **C-RD-SMOOTH-7** | Agent **≥4.9.110**: never stamp provisional `dxgi:pending`. Unknown lock (`session_locked is None`) must **not** unlock Default (prefer Winlogon). `prefer_raw` only after WebRTC media ready when `preferred_transport=webrtc`. |
| **C-RD-SMOOTH-8** | Honor Start `preferred_transport: "websocket"` (cloud default). Keep JPEG-WS on agent WS while WebRTC is connecting/connected; do **not** suppress healthy JPEG for ICE. WebRTC-primary Start may suppress JPEG only after `_media_ready` (healthy + ICE/DTLS). Agent **≥4.9.111**. |
| **C-RD-SMOOTH-9** | Agent **≥4.9.113**: keep `wss://…/ws/remote/agent` connected between streams (`websocket:true`). `gdi+flat` / var=0 never claims `phase=live`. WebRTC `peer setup failed` must not stop JPEG-WS. Wire `jpeg_fallback_active` means “JPEG path active” (true under websocket-primary). |

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
    "webrtc": { "available": true, "signaling": 1, "ice": "non-trickle", "needs_turn": true, "preferred_ice": "turns" }
  }
}
```

`t:meta` before binary JPEG (and ≥ every 5th frame): geometry, `session_id`,
`desktop`, `capture_method`, `black_frame`, `frame_variance`, `bright_ratio`,
`logonui_hwnd_count`, `inputs_applied`, adaptive fps/quality, optional
`capture_diag` (≥4.9.107). After follow, meta **must** change (FOLLOW-8).
**C-RD-PIX-7.** **C-RD-DIAG-1.**

`t:capture_diag` (agent → viewer WS, protocol 2): one-shot / on fail / on
`switching` — same fields as DIAG-2. Dashboard Capture health consumes this.

`t:stream_progress` (aliases `remote_progress` / `progress`): `phase` =
`running` | `capturing` | `ws` | `webrtc` | `switching` | `live` | `degraded` | `failed`.

Input v2: piggyback on frame ACK; move may coalesce; `mousedown`/`key*` never dropped
behind move flood.

---

## Acceptance (lab) — **client** ticks [`CLIENT_CHECKLIST.md`](./CLIENT_CHECKLIST.md)

Cloud dashboard RD P0 is done (**1.4.72** / prod 2026-08-17: Connect not auto,
follow Start without SID, honesty banner, Start 30/72/1920, TURNS:443,
JPEG-WS rAF). Remaining boxes are **agent + lab host**.

### Host prep (do not skip)

1. Note agent version on Derin-Web (or lab). **&lt;4.9.95** cannot close TOPO. **&lt;4.9.100** cannot close PIX / FOLLOW-9/10. **4.9.103 string without Run A–F PASS** cannot close PIX (2026-08-17 lab still `gdi+black`). Retest on **≥4.9.110** (JPEG-WS primary + lock probe + diag).
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

### Run D — follow after Enter **and** after lock/logoff (FOLLOW-1…10)

- [ ] From Run A: type password on the stream, Enter.
- [ ] Same `stream_id`. ≤2s freeze, `phase=switching` then `live` Default shell.
- [ ] Input after logon hits Default only (no 4× keys).
- [ ] Win+L or logoff on the **same** stream: LogonUI/lock chrome returns (not `gdi+black`). No second Start.

### Run E — ICE honesty

- [ ] Force ICE fail (or wait reject): JPEG-WS chrome ≤2s, no zombie connected.

### Run F — smoothness (C-RD-SMOOTH) · ≥4.9.101 / primary ≥4.9.110

- [ ] Start params include `fps≥30`, `quality≥72`, `max_width=1920` (not 12/40/1280).
- [ ] ICE fail or TCP-only 443: JPEG-WS stays **≥24 fps** on a gigabit/100 Mbit viewer (not ~8 fps). Viewer paints from first JPEG — no endless “Kanal / connecting”.
- [ ] If TURNS:443 is in `ice_servers`, WebRTC may connect; otherwise JPEG-WS is the video path.
- [ ] Capture health never shows stuck `dxgi:pending`; lock/unknown → `persistent-winlogon-helper` (not `persistent-user-helper` + black).
- [ ] Keys / clicks on viewer WS apply immediately (no HTTP-poll-only feel).

Historical ticks (do not regress):

- [x] Session-0 empty host Logon pixels path **existed** on **4.9.84** (re-verify PIX on current build — 2026-08-15 lab **regressed** to gdi+black)
- [x] CAD meta honesty (**4.9.86**)
- [x] Cloud Start shapes / CAD / Live honesty ([`../cloud/CLOUD_CHECKLIST.md`](../cloud/CLOUD_CHECKLIST.md))

---

## Out of scope

Hypervisor / iLO / noVNC; auto-typing credentials; Session-0 service desktop as
a fake Winlogon; publishing extra installer names (that is self-update).
