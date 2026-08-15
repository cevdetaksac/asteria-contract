# Feature contracts

> Contract **≥ 1.4.60**. One product feature → **one MD**. Client and cloud
> read **that file only**. Do not add new MUST IDs under `agent/` + `cloud/` +
> `api/` for the same feature.

## Why

Scatter (Remote Desktop example) was 10+ files: `api/05`, `agent/remote-*`,
`agent/winlogon-*`, `cloud/REMOTE_DESKTOP_*`, `cloud/remote-console-parity.md`.
Lab P0 lived in a paste (`CLOUD_HANDOFF`) while older C-WL still said
“always send prefer=winlogon”. Teams implemented different files.

## Rules

1. **New requirement** → edit the feature MD, then `CHANGELOG` + `VERSION`.
2. Old paths stay as **pointers** (no 404). If they conflict, **feature file wins**.
3. `api/01`–`03` stay shared (auth, account, control-WS catalog). Feature files
   own command *behavior*; `03` only lists `command_type` names.
4. `INDEX.md` / `FLEET.md` link the feature file, not the scatter.

## Map

| Feature | SoT | Status |
|---------|-----|--------|
| **Remote Desktop** | [`remote-desktop.md`](./remote-desktop.md) | **1.4.60** folded |
| Self-update | planned: `04-self-update` + `self-update-progress` | scatter |
| Process inspect / server mgmt | planned | scatter |
| Threat intel | planned: `09` + ingest | scatter |
| Firewall / intel blocks | planned | scatter |
| Defense policy | planned | scatter |
| Network Guard | planned | scatter |
| Anti-brick / account | planned | scatter |
| Auth / heartbeat | stays [`../api/01-auth.md`](../api/01-auth.md) | shared |
| Control WS catalog | stays [`../api/03-control-websocket.md`](../api/03-control-websocket.md) | shared |

Next fold: **self-update** (P0-3/P0-4 live in `api/04` until then).
