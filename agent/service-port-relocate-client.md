# Service Port Relocate — Client Implementation (bidirectional sync)

> **Contract VERSION:** **1.4.45**  
> Status: **Normative (client ≥ 4.9.45)**  
> Supersedes / extends: [`service-port-relocate.md`](./service-port-relocate.md) (1.4.44)  
> Related: [`attacks-and-services.md`](./attacks-and-services.md) · [`gui-control-center.md`](./gui-control-center.md)

## Goal

Cloud dashboard and **client GUI** stay in sync for real-service port moves:

| Operator action | Expected cloud view |
|-----------------|---------------------|
| Dashboard → Relocate to N | Client moves service → cloud shows relocated → N; bait Start enabled |
| Client GUI → Relocate to N | Same cloud state within seconds (no 5‑min wait) |
| Either side starts bait on classic port | Existing `tunnel_start` / desired reconcile |

**Ports are dynamic.** Defaults are 4XXXX; the operator’s saved preference (dashboard input or GUI) is the source of truth for the next relocate.

---

## Default safe ports (fallback only)

| Service | Classic | Default safe |
|---------|---------|--------------|
| RDP | 3389 | **43389** |
| MSSQL | 1433 | **41433** |
| MYSQL | 3306 | **43306** |
| SSH | 22 | **40022** |
| FTP | 21 | **40021** |

Client MUST prefer, in order:

1. Cloud `relocate_state.<SVC>.saved_target_port` (from `GET /api/premium/tunnel-status`)
2. Local GUI last-entered value (persisted locally OK)
3. `default_safe_port` from the same response
4. Hardcoded table above

Never assume 53389 / 9XXXX — those are obsolete.

---

## A. Handle cloud command: `relocate_service`

Queued by dashboard `POST /api/premium/relocate-service`.

```json
{
  "command_type": "relocate_service",
  "params": {
    "service": "RDP",
    "target_port": 43389,
    "confirm": true,
    "auto_start_bait": true
  }
}
```

### Execution (unchanged from 1.4.44)

1. Pre-check classic in use / target free  
2. Golden snapshot to disk  
3. Firewall allow target  
4. Change config → restart service  
5. Verify bind ≤10s else **local rollback**  
6. Report result (below)

### Result: `POST /api/commands/result`

On success:

```json
{
  "command_id": "<id>",
  "status": "completed",
  "result": {
    "status": "ok",
    "service": "RDP",
    "old_port": 3389,
    "new_port": 43389
  }
}
```

On rollback:

```json
{
  "command_id": "<id>",
  "status": "failed",
  "result": {
    "status": "rollback",
    "service": "RDP",
    "old_port": 3389,
    "target_port": 43389,
    "reason": "bind_verify_failed"
  }
}
```

Cloud **persists** `settings_json.service_ports` on this result and may queue `tunnel_start` when `auto_start_bait=true`.

**Also required after success or rollback:** push fresh inventory immediately (see §C).

---

## B. Client GUI relocate (local initiate)

GUI must offer the same flow as the dashboard Relocate column:

- Prefill target from `relocate_state` (see §D)
- Block start if target busy (local bind/`open_ports` check)
- Confirm destructive restart
- Run the **same** relocate pipeline as the command handler (golden + rollback)

### Report: `POST /api/agent/relocate-report` (Bearer)

```json
{
  "service": "RDP",
  "status": "ok",
  "old_port": 3389,
  "new_port": 43389,
  "source": "gui",
  "auto_start_bait": true,
  "open_ports": [
    { "port": 43389, "protocol": "tcp", "process": "svchost.exe", "pid": 776, "state": "LISTEN" }
  ]
}
```

| Field | Required | Notes |
|-------|----------|--------|
| `service` | ✓ | `RDP` / `MSSQL` / `MYSQL` / `SSH` / `FTP` |
| `status` | ✓ | `ok` \| `rollback` \| `error` |
| `old_port` | recommended | Classic or previous |
| `new_port` | ✓ on ok | Alias: `target_port` |
| `source` | | `gui` (default) \| `local` |
| `auto_start_bait` | | If true and status=ok, cloud queues bait start |
| `open_ports` | strongly recommended | Full or delta LISTEN snapshot — updates dashboard conflict UI immediately |

**200:** `{ status, relocate_status, relocated_to, bait_queued, bait_command_id }`

If the relocate was triggered by a **cloud command**, still prefer `commands/result` for that command_id; optionally also call `relocate-report` with `source=command` only if you need to attach `open_ports` in the same call (not required if §C is done).

---

## C. Immediate open-ports refresh

After every relocate attempt (ok / rollback / error that may have changed listeners):

1. Include `open_ports` on `relocate-report`, **or**
2. Call existing `POST /api/agent/open-ports` within **≤5 s**

Do **not** wait for the normal ~5 min cadence — dashboard conflict badges depend on this.

---

## D. Read cloud state for GUI sync

`GET /api/premium/tunnel-status` (Bearer) includes:

```json
{
  "relocate_state": {
    "RDP": {
      "classic_port": 3389,
      "default_safe_port": 43389,
      "saved_target_port": 43389,
      "current_port": 3389,
      "relocated": false,
      "relocated_at": null,
      "relocating": false,
      "port_available": true,
      "last_status": null,
      "last_reason": null
    }
  },
  "services": { "...": "existing + port_conflict" }
}
```

### GUI rules

| Field | Use |
|-------|-----|
| `saved_target_port` | Prefill Relocate input |
| `default_safe_port` | Fallback if no saved pref |
| `relocated` / `current_port` | Show “service on N” badge |
| `relocating` | Spinner; disable double-submit |
| `port_available` | Soft hint (cloud inventory may be stale — still do local bind check) |
| `port_conflict` on `services.<SVC>` | Show busy / Info vs Start |

When operator edits target port in GUI, optionally persist locally; cloud preference updates when relocate runs or when dashboard saves via `relocate-port-save` (dashboard-only). GUI is **not** required to call `relocate-port-save`.

---

## E. After relocate — bait

- If `auto_start_bait` was true, cloud may already queue `tunnel_start`. Honor `services[].desired`.
- If false, operator starts bait from dashboard or GUI via existing tunnel APIs.
- Bait listens on **classic** port only after real service left it.

---

## Safety (normative)

| ID | Rule |
|----|------|
| **C-REL-1** | Rollback is local; no cloud dependency |
| **C-REL-2** | Golden snapshot on disk before config mutate |
| **C-REL-3** | Serialize relocates (one at a time per host) |
| **C-REL-4** | Verify bind ≤10s or rollback |
| **C-REL-5** | Firewall rule before restart; remove on rollback |
| **C-REL-6** | Reject ports outside 1024–65535 |
| **C-REL-7** | Do not relocate onto another service’s classic port |
| **C-REL-8** | Report success/failure to cloud (`commands/result` and/or `relocate-report`) |
| **C-REL-9** | Push open_ports ≤5s after attempt |

---

## Acceptance (client)

- [ ] `relocate_service` RDP → 43389 succeeds; `commands/result` with `status:ok`, `new_port:43389`
- [ ] Failure path rolls back; cloud gets `rollback` / failed
- [ ] GUI relocate → `POST /api/agent/relocate-report` → dashboard shows Relocated → N without waiting 5 min
- [ ] GUI prefill uses `relocate_state.saved_target_port` / `default_safe_port` (4XXXX)
- [ ] open_ports refreshed within 5s after relocate
- [ ] `auto_start_bait` → bait desired/start honored when classic port free
- [ ] Min client version advertised ≥ **4.9.45**

---

## Cloud already shipped (do not reimplement)

| Piece | Path |
|-------|------|
| Queue relocate | `POST /api/premium/relocate-service` |
| Save pref (dashboard) | `POST /api/premium/relocate-port-save` |
| State for UI | `GET /api/premium/tunnel-status` → `relocate_state` |
| Persist on command result | `POST /api/commands/result` → `service_ports` |
| Persist on GUI report | `POST /api/agent/relocate-report` |
| Auto bait after ok | cloud queues `tunnel_start` when requested |
