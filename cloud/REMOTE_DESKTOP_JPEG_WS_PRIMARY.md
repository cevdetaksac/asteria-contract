# Cloud MUST — Remote Desktop Live (contract 1.4.77 / agent ≥4.9.110)

Paste into the cloud/dashboard PR. SoT: [`../features/remote-desktop.md`](../features/remote-desktop.md).

## Why

Operator Live must feel like **watching video** + **instant keyboard/mouse**.
Bandwidth is not the bottleneck (100 Mbit / multi-Gbit). Cloudflare often
blocks host UDP → WebRTC ICE stalls. Waiting on ICE while the agent already
sends JPEG over the tunnel makes Live look broken (“Kanal / connecting”).

**JPEG-WS on the agent outbound WS is the primary continuous video path.**
WebRTC is an optional upgrade after ICE/DTLS `connected`. Wire field
`fallback:"jpeg-ws"` is legacy naming — do **not** show “fallback / degraded”
copy when painting JPEG-WS.

## MUST (dashboard / remote hub)

1. **Pin** recommend agent **≥ 4.9.110** (banner, frozen-frame overlay, Capture health upgrade hint). Replace lingering ≥4.9.107 / 4.9.105 strings.
2. **Paint Live from first healthy JPEG-WS frame.** Do not gate the player on WebRTC `connected`. Progress may show `ws` / `webrtc` but the surface must already show chrome.
3. **ICE stall / reject:** keep painting JPEG-WS at video rate on the **same** surface (`requestAnimationFrame` latest-frame). Clear zombie “WebRTC connected” UI (C-RD-5 / VIEW-7).
4. **Start knobs unchanged:** every `remote_stream_start` sends `fps≥30`, `quality≥72`, `max_width:1920` (SMOOTH-1 — already prod).
5. **Input:** viewer → `wss://…/ws/remote/view` `t:input` (protocol 2) immediately; do not rely on HTTP poll for feel.
6. **Capture health:** treat stuck `dxgi:pending` / `persistent-user-helper` + `black_frame` as FAIL honesty (agent ≥4.9.110 should not stamp `dxgi:pending`).
7. **Topology Start shapes unchanged** (TOPO-1 follow / TOPO-2 winlogon row).

## Nice

- When WebRTC connects, swap to `<video>` in place without tearing down the stream_id.
- Show encode fps/quality from `t:meta` so operators see ≥24 fps.

## Not cloud

- Winlogon vs Default capture, helper spawn, `prefer_raw` — agent only.
- NETWORK attack instant-mail — see [`ATTACKS_NOTIFY_DUAL_CHANNEL.md`](./ATTACKS_NOTIFY_DUAL_CHANNEL.md) (digest, not Live).
