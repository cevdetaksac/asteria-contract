# Cloud MUST — Remote Desktop Live (contract 1.4.80 / agent ≥4.9.113)

Paste into the cloud/dashboard PR. SoT: [`../features/remote-desktop.md`](../features/remote-desktop.md).

## Why

Operator Live must feel like **watching video** + **instant keyboard/mouse**.
Bandwidth is not the bottleneck (100 Mbit / multi-Gbit). Cloudflare often
blocks host UDP → WebRTC ICE stalls. Waiting on ICE while the agent already
sends JPEG over the tunnel makes Live look broken (“Kanal / connecting”).

**JPEG-WS on the agent outbound WS is the primary continuous video path.**
Start MUST send `preferred_transport: "websocket"`. WebRTC is an optional
upgrade after ICE/DTLS `connected`. Wire field `fallback:"jpeg-ws"` / `jpeg_fallback_active` means JPEG path active — do **not** show “fallback / degraded / webrtc preferred” when websocket-primary.

## MUST (dashboard / remote hub)

1. **Pin** recommend agent **≥ 4.9.113** (persistent agent WS + JPEG honesty; Capture health).
2. Expect `/ws/remote/agent` **always up** (`websocket:true`) between streams — not only while Live.
3. **Start** includes `preferred_transport: "websocket"` (+ WebRTC second).
4. **Paint Live from first healthy JPEG-WS frame.** Do not gate the player on WebRTC `connected`. Flat/`var=0` / `gdi+flat` ≠ Live.
5. **ICE stall / peer setup failed:** keep painting JPEG-WS; never treat peer fail as stream death.
6. **Start knobs:** `fps≥30`, `quality≥72`, `max_width:1920`.
7. **Input:** viewer → `wss://…/ws/remote/view` `t:input` immediately.
8. **ICE URLs:** `stun/turn/turns` on `turn.asteria.run` (grey cloud / DNS only).
9. **Capture health:** show `root_cause`, `faults[]`, `blame` from `capture_diag` (agent ≥4.9.112). `dxgi:pending` / helper+black/flat = FAIL.

## One-liner

Ship ≥4.9.113; keep agent WS up (`websocket:true`); honor websocket-primary; peer fail must not kill JPEG; flat≠Live; use turn.asteria.run ICE URLs.
