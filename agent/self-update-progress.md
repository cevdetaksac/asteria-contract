# Self-Update progress ticks (download % + phases)

> **Contract VERSION:** **1.4.46**  
> Status: **Normative (client ≥ 4.9.60 recommended; min ≥ 4.9.54 with progress)**  
> Extends: [`api/04-self-update.md`](../api/04-self-update.md)  
> Pattern: same honesty model as [`remote-stream-progress.md`](./remote-stream-progress.md)

## Problem

Dashboard shows **“Command received — updating…”** after early ACK (`status=running`)
and can stay there for a long time (large installer ~100+ MB) or forever if download
stalls. Operators need **phase + percent** feedback, and cloud UI must time out a
silent download.

Coarse states in 04-self-update (`waiting` / `running` / `installing` / `done`) remain.
This doc adds **progress ticks** while `running`.

---

## Agent → cloud: progress via `POST /api/commands/result`

Same `command_id` as the `self_update` command. Multiple results = **merge** (already supported).

### Cadence

| Rule | Value |
|------|--------|
| **C-UPD-PROG-1** | While downloading, emit a progress result at least every **2 s** (max every **3 s**) |
| **C-UPD-PROG-2** | Never stay silent > **5 s** while status is `running` without a progress tick or terminal result |
| **C-UPD-PROG-3** | On phase change, emit immediately (do not wait for the 2 s timer) |
| **C-UPD-PROG-4** | `progress_pct` is 0–100 integer; omit or `null` only if unknown (prefer estimate from bytes) |

### Payload (additive)

```json
{
  "command_id": "<same>",
  "status": "running",
  "result": {
    "ok": true,
    "message": "update_accepted",
    "phase": "downloading",
    "progress_pct": 42,
    "bytes_done": 48000000,
    "bytes_total": 113600000,
    "detail": "download",
    "from_version": "4.9.54",
    "to_version": "4.9.60",
    "tag": "v4.9.60"
  }
}
```

### `phase` enum

| phase | Meaning | Typical pct |
|-------|---------|-------------|
| `queued` | Command accepted, not downloading yet | 0 |
| `downloading` | HTTP(S) download in progress | 1–89 |
| `verifying` | size / sha256 check | 90–94 |
| `installing` | Silent installer / helper launched | 95–99 |
| `restarting` | Waiting for new process | 99 |
| `done` | Use terminal `status=completed` instead | 100 |
| `failed` | Use terminal `status=failed` instead | — |

Terminal results stay as in 04-self-update (`completed` / `failed` / `already_current`).
When helper launches, prefer:

```json
{
  "status": "completed",
  "result": {
    "message": "update_started",
    "phase": "installing",
    "progress_pct": 95,
    "restart_required": true,
    "from_version": "…",
    "to_version": "…",
    "tag": "v…"
  }
}
```

Cloud UI then waits for heartbeat `agent_version` == target (existing behavior).

---

## Optional: control WS progress (nice-to-have)

If control WS is up, agent MAY also push:

```json
{
  "t": "update_progress",
  "command_id": "<id>",
  "phase": "downloading",
  "progress_pct": 42,
  "bytes_done": 48000000,
  "bytes_total": 113600000
}
```

Cloud may relay to dashboard viewers later. **HTTP `commands/result` remains mandatory**
so poll-only dashboards work (C-UPD-PROG-HTTP).

---

## Cloud / dashboard (shipped with 1.4.46)

| Behavior | Detail |
|----------|--------|
| Poll | `GET /api/commands/{id}` returns merged `result` including `phase`, `progress_pct`, bytes |
| UI | Progress bar + “Downloading 42% (48 / 114 MB)” under the update badge |
| Stuck | If `running` and **no result update for 120 s** → show stuck / Retry |
| Max download | If `running` > **15 min** without `completed`/`failed` → failed timeout |
| Silence | Client without progress still shows spinner (legacy); stuck timeout still applies |

---

## UI copy keys

| key | EN | TR |
|-----|----|----|
| `update_phase_downloading` | Downloading… | İndiriliyor… |
| `update_phase_verifying` | Verifying… | Doğrulanıyor… |
| `update_phase_installing` | Installing… | Kuruluyor… |
| `update_phase_restarting` | Restarting agent… | Agent yeniden başlıyor… |
| `update_progress_pct` | {pct}% | %{pct} |
| `update_progress_bytes` | {done} / {total} | {done} / {total} |
| `update_stuck` | Update stalled — no progress | Güncelleme takıldı — ilerleme yok |

---

## Acceptance

- [ ] During download, dashboard % moves at least every ~3 s when client ≥ progress build
- [ ] Phase label changes: downloading → verifying → installing
- [ ] Silent >120 s while running → stuck UI + Retry (not infinite spinner)
- [ ] Legacy client (ACK only, no pct): still shows Updating…; stuck timeout still fires
- [ ] Final version match → Up to date (unchanged)

---

## Client prompt summary

Implement progress ticks on existing `commands/result` for `self_update`. Do not invent a new endpoint. Prefer `phase` + `progress_pct` + bytes. Cadence ≤3 s while downloading. Min client advertise progress support in version notes (≥4.9.60).
