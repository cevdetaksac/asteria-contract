# Presence — single contract

> **SoT** **≥ 1.4.62**. Pointer: [`../api/11-presence-realtime.md`](../api/11-presence-realtime.md)

Sleep / shutdown / uninstall must not leave a green “online” badge.
`presence` suspend + `goodbye` on control WS; wake reconnects. Client ≥ **4.9.8**.
