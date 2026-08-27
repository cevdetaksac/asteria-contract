# Cloud MUST — Remote Desktop Live (contract 1.4.78 / agent ≥4.9.111)

Paste into the cloud/dashboard PR. SoT: [`../features/remote-desktop.md`](../features/remote-desktop.md).

## Why

Operator Live must feel like **watching video** + **instant keyboard/mouse**.
Bandwidth is not the bottleneck (100 Mbit / multi-Gbit). Cloudflare often
blocks host UDP → WebRTC ICE stalls. Waiting on ICE while the agent already
sends JPEG over the tunnel makes Live look broken (“Kanal / connecting”).

**JPEG-WS on the agent outbound WS is the primary continuous video path.**
Start MUST send `preferred_transport: "websocket"`. WebRTC is an optional
upgrade after ICE/DTLS `connected`. Wire field `fallback:"jpeg-ws"` is legacy
naming — do **not** show “fallback / degraded” copy when painting JPEG-WS.

## MUST (dashboard / remote hub)

1. **Pin** recommend agent **≥ 4.9.111** (banner, frozen-frame overlay, Capture health).
2. **Start** includes `preferred_transport: "websocket"` (+ WebRTC second).
3. **Paint Live from first healthy JPEG-WS frame.** Do not gate the player on WebRTC `connected`.
4. **ICE stall / reject:** keep painting JPEG-WS at video rate on the **same** surface.
5. **Start knobs:** `fps≥30`, `quality≥72`, `max_width:1920`.
6. **Input:** viewer → `wss://…/ws/remote/view` `t:input` immediately.
7. **ICE URLs:** `stun/turn/turns` on `turn.asteria.run` (grey cloud / DNS only).
8. **Capture health:** show `root_cause`, `faults[]`, `blame` from `capture_diag` (agent ≥4.9.112). `dxgi:pending` / helper+black/flat = FAIL.

## One-liner

Ship ≥4.9.111; honor websocket-primary Start; never suppress healthy JPEG for ICE; dxgi:pending / helper+black = FAIL; use turn.asteria.run ICE URLs from cloud.
