# Remote stream progress — live stage events (dashboard honesty)

> **Contract VERSION:** **1.4.39**  
> Status: **Normative (client ≥ 4.9.38 target)**  
> Related: [`remote-desktop-p0.md`](./remote-desktop-p0.md) ·
> [`../api/05-remote-desktop.md`](../api/05-remote-desktop.md) ·
> [`REMOTE_DESKTOP_SMOOTHNESS.md`](./REMOTE_DESKTOP_SMOOTHNESS.md)

## Problem

Operators click **Bağlan** and see a sticky “Starting…” / “Connecting…” label
with no proof whether:

1. Cloud queued the command  
2. Agent **acked** it (`running`)  
3. Capture / logon actually started  
4. First frame is on the wire  

HTTP command polling helps, but **WebSocket progress** is the low-latency path
we already prefer for RD.

## Wire (agent → cloud → viewer)

Agent sends JSON text on **`/ws/remote/agent`**. Cloud relays to
**`/ws/remote/view`** viewers unchanged (kinds listed below).

```json
{
  "t": "stream_progress",
  "protocol": 1,
  "stream_id": "…",
  "command_id": "…",
  "phase": "running",
  "message": "Attaching Winlogon desktop…",
  "ts": 1710000000123
}
```

Aliases accepted by cloud/dashboard: `t: "remote_progress"` | `"progress"`.

### `phase` vocabulary (client SHOULD use these)

| phase | Meaning | Dashboard step |
|-------|---------|----------------|
| `queued` / `pending` | Local queue before work | 1 · Queue |
| `dispatched` / `waiting` / `ack_wait` | Waiting for execution start | 2 · Agent |
| `running` / `logon` / `prepare` / `capture_start` | Doing work | 2 · Agent |
| `capturing` / `encoding` / `streaming` | Producing media | 3 · Channel |
| `ws` / `channel_open` | Agent media WS up | 3 · Channel |
| `webrtc` / `ice` / `dtls` | WebRTC path | 3 · Channel |
| `live` / `connected` | First real frame / media up | 4 · Live |
| `failed` / `error` | Terminal failure (`error` field) | fail |

Emit **at least**:

1. When `remote_stream_start` / `remote_session_prepare` is **received** → `running`  
2. When capture attach begins → `capture_start`  
3. Before first JPEG/WebRTC frame → `capturing`  
4. On hard failure → `failed` + `error` code (same codes as command result)

Rate-limit: ≤ **4 events/s**; coalesce noisy ICE ticks.

## Client MUST (C-RD-PROG-*)

| ID | Rule |
|----|------|
| **C-RD-PROG-1** | During `remote_stream_start` and `remote_session_prepare`, emit `stream_progress` on agent RD WS (or control WS if RD agent socket not yet up — cloud may only relay RD agent WS today) |
| **C-RD-PROG-2** | Never stay silent > **3 s** while status is `running` without a progress tick or terminal result |
| **C-RD-PROG-3** | `failed` progress MUST include `error` matching command result (`CAPTURE_NO_DESKTOP`, `AUTH_FAILED`, …) |
| **C-RD-PROG-4** | Do not emit `live`/`connected` for black-fill-only frames (see P0-A) |

## Cloud (shipped with 1.4.38 dashboard)

- Relays `stream_progress` / `remote_progress` / `progress` agent → viewers  
- Dashboard shows a **4-step pipe** + elapsed seconds from command poll ticks
  even when the client has not yet implemented this doc

## Acceptance

- [ ] Lab: Bağlan → UI moves Queue → Agent(`running`) within 2s of agent ack  
- [ ] Lab: kill agent mid-start → `failed` or timeout copy (not eternal “Starting”)  
- [ ] Lab: with progress events, subtitle updates without waiting for HTTP poll  
