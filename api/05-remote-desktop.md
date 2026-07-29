# Remote Desktop

> **Canonical** cloud / dashboard contract for the Windows client Remote
> Desktop feature. This file is the single source of truth; the prompt-sourced
> material below the v2 section is **legacy** and retained for history only.
> API: `https://asteria.run`
> Base client: **≥ 4.5.48** (`list_local_users` / `remote_session_prepare`).
> **Remote Desktop v2: client ≥ 4.9.0** (see next section).
> Companion: [`../agent/remote-input.md`](../agent/remote-input.md) (input path).

---

# ▶ Remote Desktop v2 (client ≥ 4.9.0)

> **Audience:** cloud + dashboard implementers.
> Everything in this delimited section reflects the **actual** client 4.9.0
> implementation (`client_remote_desktop.py`, `client_rd_session_helper.py`,
> `client_rd_adaptive.py`, `client_rd_media.py` and their tests). Fields are
> exact. Where a v2 statement conflicts with the legacy prompt material further
> down, **v2 wins** and the legacy line is marked *(superseded by v2)*.

## v2 at a glance

- **Transports, in priority order:** WebRTC (optional, only if the client
  advertises it) → **JPEG over WebSocket** (healthy default) → JPEG over HTTP
  (fallback). The healthy path is **WS + JPEG only** — a healthy stream does
  **not** also POST every frame over HTTP.
- **Endpoints unchanged.** Still `wss://…/ws/remote/agent` (agent) and
  `wss://…/ws/remote/view` (viewer), plus HTTP `POST /api/remote/frame`,
  `POST /api/remote/frame-json`, `GET /api/remote/inputs`, `POST /api/remote/input`,
  `POST /api/remote/session`, `GET /api/remote/status`, `POST /api/remote/cad`.
  No new endpoint is required for v2. WebRTC signaling is **relayed over the
  existing agent/view WS**. Any genuinely new endpoint is flagged **Cloud TODO**.
- **Two protocol numbers, do not confuse them:**
  - `protocol: 2` → WS **application** envelope (`hello`, `meta`, `input`,
    `input_ack`).
  - `protocol: 1` → **WebRTC signaling** envelope (`webrtc_offer` / answer /
    reject / ice). The client rejects signaling with any other protocol value.
- **Honesty invariants (unchanged from v1):** `streaming:true` is never faked;
  no interactive desktop → `NO_INTERACTIVE_SESSION`; 0×0 / black capture →
  `CAPTURE_NO_DESKTOP`; frames `< 1500 B` are rejected as too small.
- **Console / Logon screen (Winlogon):** Start with `prefer=winlogon` +
  `pre_logon=true` + `desktop=Winlogon`, **no username**. Normative rules:
  [`../cloud/REMOTE_DESKTOP_WINLOGON.md`](../cloud/REMOTE_DESKTOP_WINLOGON.md) +
  physical-console acceptance
  [`../cloud/remote-console-parity.md`](../cloud/remote-console-parity.md)
  (≥**1.4.43**, client ≥**4.9.49**). Omit `session_id` → agent resolves active
  console SID (never assume `1`).

## 1. Agent WS `hello` (agent → server, once on connect)

Sent as the first text frame after `wss://…/ws/remote/agent` connects
(Authorization: `Bearer <token>`; legacy `?token=` only if the client has
`api.legacy_token_query` enabled). **Exact, additive schema:**

```json
{
  "t": "hello",
  "role": "agent",
  "protocol": 2,
  "stream_id": "6f1c…hex",
  "capabilities": {
    "input_protocols": [1, 2],
    "input_v2": true,
    "transports": ["jpeg-ws", "jpeg-http"],
    "fallback": "jpeg-ws",
    "codecs": ["jpeg"],
    "webrtc": {
      "available": false,
      "signaling": 1,
      "ice": "non-trickle",
      "ice_server_config": false
    }
  }
}
```

**Truthful capabilities (do not assume more than advertised):**

- `codecs` always begins with `"jpeg"`. Extra codecs (`"h264"`, `"vp8"`) are
  appended **only** when the optional `aiortc`/`av` runtime is actually present
  and a real `RTCPeerConnection` was constructed successfully at probe time.
- `transports` is `["jpeg-ws", "jpeg-http"]` by default; `"webrtc"` is
  **prepended** (→ `["webrtc", "jpeg-ws", "jpeg-http"]`) only when the runtime
  is available.
- `webrtc.available` is `false` on the default build (no aiortc). `webrtc.signaling`
  is always `1`; `webrtc.ice` is always `"non-trickle"`.
- `webrtc.ice_server_config` is `true` **only** when the WebRTC runtime is
  available — it advertises that the client accepts cloud-supplied
  `ice_servers` inside the protocol-1 offer (see §5). On the default build it
  is `false`; a viewer must not attach `ice_servers` to an offer unless this
  flag is `true`.
- `fallback` is always `"jpeg-ws"`.

Cloud **must** gate any WebRTC offer on `capabilities.webrtc.available === true`
**and** `"webrtc" ∈ transports`. Otherwise use JPEG-over-WS.

## 2. Agent WS `meta` (agent → server, before every binary frame)

A `meta` text frame is enqueued **immediately before each binary JPEG** (and at
minimum every 5th frame). It gives the viewer the geometry and adaptive state
for the frame that follows. **Exact schema:**

```json
{
  "t": "meta",
  "protocol": 2,
  "stream_id": "6f1c…hex",
  "capabilities": { "...": "same object as hello" },
  "width": 1280,
  "height": 720,
  "native_width": 1920,
  "native_height": 1080,
  "origin_x": -1920,
  "origin_y": 0,
  "seq": 42,
  "fps": 6.0,
  "quality": 35,
  "max_width": 1280,
  "requested_fps": 8.0,
  "requested_quality": 40,
  "requested_max_width": 1600,
  "capture_mono_ms": 123456,
  "last_send_mono_ms": 123450,
  "session_id": 2,
  "username": "Administrator",
  "media": { "...": "media transport status, see §6" }
}
```

**Dimension semantics (critical for correct pointer mapping):**

- `width` / `height` = **encoded** (downscaled) JPEG dimensions actually sent.
- `native_width` / `native_height` = the **source monitor** resolution.
- `origin_x` / `origin_y` = the captured monitor's **virtual-desktop origin**
  and **can be negative** (secondary monitor left of / above primary).
- Normalized input `x,y ∈ [0,1]` maps to a device pixel as
  `px = origin + x·(native−1)` (see §4). The viewer must send normalized
  coordinates against the **native** rectangle, not the encoded one.

**Adaptive telemetry** exposes both the client-**requested** ceiling
(`requested_fps/quality/max_width`) and the current **effective** values
(`fps/quality/max_width`), so the dashboard can render "8 fps requested → 6 fps
effective" style badges. `capture_mono_ms` / `last_send_mono_ms` are monotonic
millisecond stamps for latency/liveness display.

## 3. Binary frames + latest-frame / coalescing semantics

- Each video frame is a **raw JPEG** binary WS message (`FF D8 … FF D9`),
  preceded by its `meta` text frame.
- The outbound queue keeps **control/meta messages in order** but retains **only
  the newest JPEG**. If a new frame is produced before the previous one is sent,
  the stale frame is dropped (coalesced) and counted in `stats.frames_coalesced`
  — the viewer never receives a backlog of stale frames.
- Frame accounting (`stats.frames_sent`, `bytes_sent`) increments **only on the
  actual socket send**, never on enqueue.
- On (re)connect the client re-sends `hello`, a forced `meta`, and the last good
  frame so the viewer is not blank while waiting for the next capture.

## 4. Healthy path vs HTTP fallback (transport decision)

Per frame the client decides (`_dispatch_frame`):

1. If a WebRTC session is **active**, the frame is published to the media track
   and `transport = "webrtc"`. No WS/HTTP frame is sent for that frame.
2. Else the frame is buffered for the WS thread. If the WS is healthy,
   `transport = "websocket"` and the WS thread performs the send. **No duplicate
   HTTP POST occurs on the healthy path** *(supersedes the legacy "POST HTTP
   frame on every WS frame" guidance below and in the old remote-input.md)*.
3. Else (WS down/unhealthy) the client uploads over HTTP and `transport = "http"`:
   - `POST /api/remote/frame` (multipart, file field **`file`**) **or**
   - `POST /api/remote/frame-json` `{ token, image_base64, width, height, seq }`.
   - The **frame ACK may carry `inputs[]`**, which the client drains and applies
     — this is the **primary input path while on HTTP**.
- `MIN_JPEG_BYTES = 1500`: the client never sends a smaller frame and expects
  the server to keep rejecting `< ~1500 B` ("Frame too small"). Nearly-black
  captures are skipped, not sent.

### Input backup cadence semantics

- **Primary input** is delivered to the agent over the **WS text channel**
  (`{"t":"input",…}`), or via **frame-ACK `inputs[]`** while the stream is on
  HTTP, or over the **WebRTC data channel** when WebRTC is active.
- `GET /api/remote/inputs?token=…&limit=80` is a **compatibility backup drain**
  that runs at **two cadences**: ~**2.0 s** while the WS is healthy (slow, to
  avoid redundant round-trips) and ~**0.30 s** while the WS is down. It is never
  the sole path and is safe to leave enabled.

## 5. WebRTC signaling over the agent/view WS relay (client ≥ 4.9.0, optional)

WebRTC is **optional** and only reachable when the client advertised it in
`hello`. Signaling is **relayed by the cloud over the existing WS endpoints**
(`/ws/remote/view` ⇄ relay ⇄ `/ws/remote/agent`); there is **no dedicated
signaling endpoint**. All signaling envelopes use **`protocol: 1`**.

**The viewer is the offerer; the client only answers.** The client never
initiates an offer.

### Offer (viewer → cloud relay → agent)

```json
{
  "t": "webrtc_offer",
  "protocol": 1,
  "stream_id": "6f1c…hex",
  "session_id": "peer-abc",
  "sdp": "v=0…complete offer SDP…",
  "ice_servers": [
    { "urls": "stun:stun.example.test:3478" },
    {
      "urls": [
        "turn:turn.example.test:3478?transport=udp",
        "turns:turn.example.test:5349?transport=tcp"
      ],
      "username": "short-lived-user",
      "credential": "short-lived-secret"
    }
  ]
}
```

`ice_servers` is **optional** and only valid on the **offer** (never on
`answer`/`ice`); when omitted, the client builds its peer connection with the
aiortc default configuration. The client also accepts
`{"t":"webrtc_signal","action":"offer",…}`. Strict validation before the offer
is handed to the media thread — the client rejects (with `webrtc_reject`)
unless **all** hold:

- `protocol === 1` (missing or `2` → rejected);
- the stream is running and has a `stream_id`;
- `stream_id` **exactly** equals the current stream's id (stale → rejected);
- `session_id` is present; once a peer is bound, later signals with a different
  `session_id` are rejected;
- `action ∈ { offer, answer, ice }`;
- the client's WebRTC runtime is available.

The first valid `offer` binds `session_id` as the media peer.

### Cloud-supplied ICE configuration (`ice_servers`, client ≥ 4.9.0)

The client **consumes** `ice_servers` from the offer and applies it to its
`RTCPeerConnection` (`RTCConfiguration`/`RTCIceServer`). Identity validation
(`protocol`/`stream_id`/`session_id`, §5 above) runs **first**; a stale offer is
rejected before its `ice_servers` are even parsed.

**Exact client validation bounds** (any violation → offer rejected via
`webrtc_reject`; error text never echoes supplied values):

- `ice_servers` must be a **list** of at most **8** server objects.
- Each server object allows **only** the keys `urls`, `username`, `credential`
  — any unknown field rejects the offer.
- `urls` is a single string or a non-empty list of at most **8** URLs;
  each URL is ≤ **512** chars, must match scheme **`stun:` / `turn:` / `turns:`**
  (case-insensitive; anything else, e.g. `https:`, is rejected), must have a
  non-empty endpoint after the scheme, and must contain no whitespace or
  control characters.
- `username` ≤ **256** chars, `credential` ≤ **512** chars (may be empty
  strings); both must be strings without whitespace/control characters.
- If `ice_servers` is absent or `null`/empty, the aiortc **default**
  configuration is used.

**Credential privacy (client-enforced, cloud must match):** validation and
peer-setup errors are deliberately generic — the client never places supplied
URLs, usernames or credentials in error messages, `webrtc_reject` payloads,
logs or status. If the runtime constructor itself fails, the viewer sees only
`"invalid ice server configuration"` / `"peer setup failed"`.

### Answer (agent → cloud relay → viewer)

```json
{
  "t": "webrtc_signal",
  "action": "answer",
  "protocol": 1,
  "session_id": "peer-abc",
  "stream_id": "6f1c…hex",
  "sdp": "v=0…complete answer SDP…",
  "type": "answer",
  "ice": "non-trickle"
}
```

### Reject (agent → cloud relay → viewer)

```json
{
  "t": "webrtc_reject",
  "protocol": 1,
  "stream_id": "6f1c…hex",
  "session_id": "peer-abc",
  "error": "stale or mismatched stream_id"
}
```

### ICE / non-trickle (important)

- Signaling is **non-trickle**: offers and answers contain the **complete SDP
  after ICE gathering**. There are no incremental candidates on the happy path.
- **The client currently does NOT accept standalone trickle ICE.** A standalone
  `ice` signal is validated but rejected with `{"reason":"non_trickle_ice"}`
  (surfaced to the viewer as a `webrtc_reject`). Cloud/dashboard must therefore
  send full-SDP offers/answers and must not rely on trickle ICE.

### Codec preference & data-channel input

- The client prefers **H264** (`setCodecPreferences` H264-first) but retains
  negotiated fallback codecs; the actually-selected codec is reported in
  `media.codec` (see §6).
- Remote input can flow over the **WebRTC data channel** using the **same
  input-v2 envelope** as WS (`{"t":"input","protocol":2,"id":…,"input":{…}}`);
  it is routed through the identical validator.

### JPEG WS fallback from WebRTC

- If the peer connection fails / disconnects / closes (or ICE fails), the client
  automatically falls back: it clears the media session and resumes JPEG frames
  over WS (`transport → "websocket"`, or `"http"` if WS is also down). The viewer
  should present a `<video>` element for WebRTC and seamlessly fall back to the
  JPEG `<img>` renderer.

### Client WebRTC smoothness (promoted 1.4.20 / ≥4.9.20)

> **Promoted (contract 1.4.20 / client ≥4.9.20):** raw RGB WebRTC path (no JPEG
> double-encode), HW H.264 when FFmpeg exposes nvenc/qsv/amf, idle frame skip,
> input fluidity. Cloud/viewer checklist:
> [../cloud/REMOTE_DESKTOP_SMOOTHNESS.md](../cloud/REMOTE_DESKTOP_SMOOTHNESS.md).

Historical note: early 4.9.x WebRTC connected but felt less fluid than commercial
RD due to JPEG staging into aiortc. Status:

> **Client ≥4.9.20:** items 1–6 addressed — JPEG staging removed on the WebRTC
> path; HW encoder preferred; static-desktop skip; input rate/ACK tightened;
> adaptive JPEG knobs no longer thrash helper while media is connected;
> media.encoder / bitrate telemetry additive.

1. **Remove the internal 10 fps request clamp** — done (≥4.9.1); JPEG ≤30;
   WebRTC capture **30–60** independent of JPEG fps.
2. **Stop JPEG WS frames while WebRTC is connected** — done (≥4.9.1).
3. **Decouple WebRTC encode from JPEG knobs** — done (≥4.9.20):
   - capture at **30–60 fps**; skip publish when frame unchanged (idle);
   - **hardware encoder** (NVENC / QuickSync / AMF) when available,
     else libx264 ultrafast+zerolatency;
   - BWE via aiortc target_bitrate; static desktop near-idle.
4. **Frame pacing** — latest-frame mailbox (raw RGB or JPEG fallback).
5. **Re-offer / reject** — done (≥4.9.1).
6. **Telemetry** — media.encoder ∈ nvenc|qsv|amf|x264|aiortc,
   effective_capture_fps, optional target_bitrate_bps.

### Cloud requirements for WebRTC

1. **Viewer-created offer:** dashboard builds the offer; cloud relays it to the
   agent with `protocol:1` + `stream_id` (current) + `session_id` (peer id).
2. **Answer/reject relay:** relay the agent's `answer` / `webrtc_reject`
   verbatim back to the originating viewer, keyed by `stream_id` + `session_id`.
3. **Runtime capability gating:** offer WebRTC **only** when the agent `hello`
   advertised `capabilities.webrtc.available` + `"webrtc" ∈ transports` (and,
   for H264, `"h264" ∈ codecs`). Attach `ice_servers` to the offer only when
   `capabilities.webrtc.ice_server_config` is `true`. Otherwise stay on
   JPEG-over-WS.
4. **Session cleanup / stale rejection:** each `remote_stream_start` creates a
   **new `stream_id`**; cloud must drop peers bound to a previous `stream_id`,
   tear down the RTCPeerConnection on stream stop, and honor client
   `webrtc_reject` for stale `stream_id`/`session_id`.
5. **TURN/STUN configuration delivery & security (client consumer implemented,
   ≥ 4.9.0):** the delivery flow rides the **existing viewer/agent WS relay
   only** — no HTTP endpoint is added:
   1. Cloud issues **short-lived** TURN credentials (e.g. per stream, TTL
      minutes not hours) and sends the ICE config to the **viewer** over the
      existing `/ws/remote/view` connection **before** the viewer creates its
      offer. The viewer-bound message is **cloud-side** (the agent never sees
      or needs it); designated shape:

      ```json
      { "t": "webrtc_ice_config", "protocol": 1, "stream_id": "6f1c…hex",
        "ice_servers": [ { "urls": "…", "username": "…", "credential": "…" } ] }
      ```

   2. The viewer constructs its own `RTCPeerConnection` with those servers
      **and** forwards the **same** `ice_servers` array inside the
      `webrtc_offer` so the agent side uses matching TURN/STUN.
   3. Cloud keeps the forwarded `ice_servers` within the client bounds above
      (≤ 8 servers, ≤ 8 URLs each, `stun|turn|turns` only, length limits) —
      an out-of-bounds config makes the agent reject the whole offer.
   4. **Credential redaction:** cloud must redact `ice_servers[].username`/
      `credential` from request/relay logs, audit trails and any status/debug
      surfaces (mirror of `scrub_command_params` practice); the client already
      guarantees it never echoes them back.
   5. STUN-only configs need no credentials; `ice_servers` stays optional and
      JPEG-WS remains the guaranteed fallback path.

## 6. `media` status object (in `hello`/`meta` and `remote_stream_start` result)

`media` mirrors the transport's live state, e.g.:

```json
{
  "available": true,
  "active": false,
  "connection_state": "new",
  "ice_state": "new",
  "codec": "",
  "preferred_codec": "h264",
  "error": "",
  "mailbox_coalesced": 0,
  "session_id": "peer-abc",
  "stream_id": "6f1c…hex"
}
```

When the runtime is absent, `available:false` and `connection_state:"unavailable"`.

## 7. `remote_stream_start` result (honest, rich)

`remote_stream_start` returns `data = get_status()` — a superset of the legacy
shape. Notable fields: `transport` (`idle|websocket|http|webrtc`), `websocket`
(bool), `requested{fps,quality,max_width}`, `effective{fps,quality,max_width}`,
`screen{x,y,w,h}` (origin + native), `capture{w,h}` (encoded), `session_id`,
`stream_id`, `username`, `monitor`, `capture_method`, `telemetry{…}` (see §8),
`media{…}` (§6), `capabilities{…}` (§1) and `stats{…}`. `screen`/`capture` `0`
still yields `success:false` + `CAPTURE_NO_DESKTOP`.

## 8. Adaptive stream telemetry (requested vs effective)

The client runs a conservative local adaptive controller
(`client_rd_adaptive.py`) bounded by the caller-requested ceilings:

- **Degrade** at most once per **5 s cooldown** under backpressure (coalesced
  frames, slow/failed sends, WS failures): `fps ×0.8`, `quality −5`,
  `max_width ×0.85`, floored at `1 fps / 20 q / 640 px`.
- **Recover** one small step (`fps +0.5`, `quality +2`, `max_width +128`, capped
  at the requested ceiling) only after a **20 s stable window**.
- `telemetry` (also under `get_status().telemetry`) exposes:
  `capture_ms_ewma`, `send_ms_ewma`, `http_ms_ewma`, `capture_samples`,
  `send_samples`, `http_failures`, `ws_failures`, `coalesced_frames`,
  `degrades`, `recovers`, `last_change_mono_ms`, plus `last_capture_mono_ms`,
  `last_send_mono_ms`.

Dashboards should show effective vs requested and a "degraded" badge when
`effective < requested`.

## 9. Cloud TODO / acceptance checklist (hand this to the cloud agent)

Ordered, concrete work items for the cloud + dashboard implementation. Existing
WS/HTTP endpoints stay; only items explicitly marked **(new)** add surface.

1. **Parse the additive `hello`/`meta` v2 fields** (`protocol`, `capabilities`,
   `requested_*`, `native_*`, `origin_*`, `capture_mono_ms`, `media`) without
   breaking legacy viewers. Ignore unknown fields forward-compatibly.
2. **Healthy-path frame handling:** accept WS-only JPEG streams; do **not**
   require or expect a duplicate HTTP frame POST per WS frame. Keep the
   `< 1500 B` "Frame too small" rejection.
3. **Pointer mapping / mobile UX:** map dashboard pointer + touch input to
   normalized `x,y` against the **native** rectangle honoring `origin_x/y`
   (support negative origins / multi-monitor). Implement the mobile pointer UX:
   tap, double-tap, long-press (→ right-click), drag (direct + trackpad/relative),
   two-finger vertical **and** horizontal scroll, and a trackpad mode. Send them
   as **input-v2** envelopes (§ remote-input.md).
4. **Input-v2 envelope + ACK:** send `{"t":"input","protocol":2,"id",…,"input":{…}}`
   over WS (and data channel when WebRTC); consume `{"t":"input_ack","protocol":2,
   "id","success"[,"error"]}`; keep accepting legacy flat protocol-1 input.
5. **WebRTC signaling relay (new plumbing over existing WS):** viewer builds the
   offer; relay `webrtc_offer`/`answer`/`reject` (`protocol:1`) between view and
   agent keyed by `stream_id`+`session_id`; gate on advertised capabilities
   (incl. `webrtc.ice_server_config` before attaching `ice_servers`);
   **do not** rely on trickle ICE (client rejects standalone ICE). Add a
   `<video>` element with automatic fallback to the JPEG renderer.
6. **Session lifecycle:** treat each `remote_stream_start` `stream_id` as
   authoritative; drop/cleanup stale peers and stream state on stop / restart;
   honor `webrtc_reject`.
7. **TURN/STUN delivery (client consumer shipped in 4.9.0 — cloud work is
   actionable now):** stand up/configure TURN+STUN; mint **short-lived**
   per-stream credentials; push `{"t":"webrtc_ice_config","protocol":1,
   "stream_id",…,"ice_servers":[…]}` to the viewer over `/ws/remote/view`
   before offer creation; viewer forwards the identical bounded `ice_servers`
   in the `webrtc_offer`; enforce the client bounds (≤8 servers / ≤8 URLs /
   `stun|turn|turns` / length limits) server-side; **redact credentials** from
   logs, audits and status surfaces. No new HTTP endpoint.
8. **Telemetry / status badges:** surface `transport`, effective-vs-requested
   fps/quality/width, `frames_coalesced`, `degrades/recovers`, `ws_failures`,
   WebRTC `connection_state`/`codec`, and a live "WS / WebRTC / HTTP" transport
   badge on `GET /api/remote/status`.
9. **Browser / mobile test matrix:** verify on desktop Chrome/Edge/Firefox/Safari
   and mobile Safari (iOS) + Chrome (Android): WS-JPEG happy path, WebRTC happy
   path + fallback, drag/scroll gestures, keyboard/Unicode input, negative-origin
   multi-monitor mapping, and reconnect after WS drop.

---

---

## Legacy notes

Pre-v2 prompt dumps were removed from this repo (non-normative). Where older notes
disagree with the v2 sections above, **v2 wins**.
