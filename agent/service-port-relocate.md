# Service Port Relocate — one-click real-service → safe port

> **Behavior SoT:** [`../features/service-port.md`](../features/service-port.md) (**≥ 1.4.61**). Appendix below.

> **Contract VERSION:** **1.4.44** (base) · extended by **1.4.45**  
> Status: **Normative** — see **[`service-port-relocate-client.md`](./service-port-relocate-client.md)** for bidirectional sync + GUI (client ≥ **4.9.45**)  
> Related: [`attacks-and-services.md`](./attacks-and-services.md) ·
> [`network-adapter-admin.md`](./network-adapter-admin.md)

## Goal

Before enabling a bait trap on a classic port (3389, 22, 1433…), the real
Windows service must move to a safe alternate port. Today this is manual —
operators must know the registry key / config file for each service.

`relocate_service` automates this: **stop service → change port config →
start service on new port → verify bind → firewall allow**. If verify fails,
**automatic rollback** to the original port.

---

## Default port map

| Service key | Classic port | Default safe port | Config location |
|-------------|-------------|-------------------|-----------------|
| `RDP`       | 3389        | **43389**         | Registry `HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp\PortNumber` |
| `MSSQL`     | 1433        | **41433**         | SQL Server Configuration Manager / Registry |
| `MYSQL`     | 3306        | **43306**         | `my.ini` / `my.cnf` `port=` |
| `SSH`       | 22          | **40022**         | `sshd_config` `Port` |
| `FTP`       | 21          | **40021**         | IIS metabase / FileZilla / config |

Operator MAY override the safe port via dashboard; cloud validates
`1024 ≤ port ≤ 65535` and port is not already in use (cross-checked against
`open_ports` inventory).

---

## Command: `relocate_service` (destructive, confirm required)

```json
{
  "command_type": "relocate_service",
  "params": {
    "service": "RDP",
    "target_port": 43389,
    "confirm": true
  }
}
```

| Field | Type | Required | Note |
|-------|------|----------|------|
| `service` | string | ✓ | One of `RDP`, `MSSQL`, `MYSQL`, `SSH`, `FTP` |
| `target_port` | int | ✓ | 1024–65535, validated free on `open_ports` |
| `confirm` | bool | ✓ | Cloud requires `true` (destructive gate) |

### Client execution flow

```
1. Pre-check
   a. Identify the running Windows service (e.g. TermService for RDP)
   b. Verify classic port is currently in use by that service (open_ports)
   c. Verify target_port is NOT in use (bind test or open_ports)
   d. If any fail → result: { status: "error", reason: "..." }

2. Snapshot current state (golden)
   a. Record: service_name, current_port, config_path, config_value
   b. Persist locally (survives crash)

3. Firewall: allow target_port inbound
   a. Add AR-RELOCATE-<SVC>-<PORT> rule (TCP inbound)

4. Config change
   a. RDP:   Set-ItemProperty PortNumber = target_port
   b. MSSQL: WMI/Registry TCP port → target_port
   c. MYSQL: my.ini port= target_port
   d. SSH:   sshd_config Port target_port
   e. FTP:   service-specific config

5. Service restart
   a. Stop-Service → Start-Service (or Restart-Service)
   b. Timeout: 30s

6. Verify (within 10s)
   a. TCP bind test on target_port → service is listening
   b. If RDP: optional quick RDP handshake to 127.0.0.1:target_port

7. SUCCESS path
   a. Remove old-port firewall rule if exclusively for this service
   b. Result: { status: "ok", service, old_port, new_port: target_port }
   c. Report to cloud via command result

8. FAILURE / ROLLBACK path
   a. Restore config from golden snapshot (step 2a)
   b. Restart service on original port
   c. Remove AR-RELOCATE firewall rule
   d. Result: { status: "rollback", service, reason, old_port, target_port }
   e. Report to cloud via command result
```

### Post-success cloud behavior

When cloud receives `status: "ok"`:
1. Update `settings_json.service_ports.<SERVICE>` = `{ classic, relocated_to, relocated_at }`
2. Mark tunnel conflict as resolved (port_conflict cleared on next poll)
3. **Optional auto-start bait:** if `auto_start_bait_after_relocate` setting
   is enabled, queue `tunnel_start` for the classic port automatically.

---

## Cloud API additions

### `POST /api/premium/relocate-service`

Dashboard → cloud. Validates and queues `relocate_service` command.

```json
{
  "token": "<client-token>",
  "service": "RDP",
  "target_port": 43389,
  "auto_start_bait": true
}
```

**Validation:**
- `service` ∈ `{RDP, MSSQL, MYSQL, SSH, FTP}`
- `1024 ≤ target_port ≤ 65535`
- `target_port` not in `open_ports` (LISTEN state, non-bait process)
- `target_port ≠ classic_port` for any known service
- Agent online (last_seen < 180s) — warn if stale

**Response:** `{ status: "queued", command_id }` or `400/409` with reason.

### `POST /api/premium/relocate-port-save`

Dashboard → cloud. Saves operator's preferred target port without starting
relocation. Persists to `settings_json.relocate_prefs.<SERVICE>`.

```json
{
  "token": "<client-token>",
  "service": "RDP",
  "target_port": 43389
}
```

Returns `{ status: "ok", port_available: true/false }` after checking
against `open_ports`.

### Additions to `GET /api/premium/tunnel-status`

Response gains:

```json
{
  "relocate_state": {
    "RDP": {
      "classic_port": 3389,
      "default_safe_port": 43389,
      "saved_target_port": 43389,
      "current_port": 3389,
      "relocated": false,
      "relocating": false,
      "port_available": true
    }
  }
}
```

---

## Dashboard UX

Each tunnel row gains a **Relocate** column (visible only when port_conflict
exists OR service not yet relocated):

| State | UI |
|-------|----|
| Port free (no conflict) | Start button (existing) |
| Conflict + not relocated | Port input (default 4XXXX) + **Relocate** button |
| Relocating in progress | Spinner + "Relocating…" |
| Relocated | ✅ badge + current port shown + Start button for bait |
| Relocate failed (rollback) | ⚠️ with reason + Retry |

Port input: auto-filled with `default_safe_port` (e.g. 43389). Operator
can edit — on blur/change, cloud checks availability via
`relocate-port-save` and shows green check / red X inline.

---

## Settings persistence

`settings_json` gains:

```json
{
  "service_ports": {
    "RDP": { "classic": 3389, "relocated_to": 43389, "relocated_at": "2026-07-29T..." },
    "MSSQL": { "classic": 1433, "relocated_to": null }
  },
  "relocate_prefs": {
    "RDP": 43389,
    "MSSQL": 41433
  }
}
```

---

## Safety invariants

| ID | Rule |
|----|------|
| **C-REL-1** | Rollback is LOCAL — does not depend on cloud connectivity |
| **C-REL-2** | Golden snapshot MUST be persisted to disk before any config change |
| **C-REL-3** | Only ONE relocate per client at a time (queued serialization) |
| **C-REL-4** | If verify fails within 10s → automatic rollback, no manual intervention |
| **C-REL-5** | Firewall rule for new port is added BEFORE service restart, removed on rollback |
| **C-REL-6** | Port 1–1023 rejected (privileged range); >65535 rejected |
| **C-REL-7** | Cannot relocate to a port used by another known service's classic or relocated port |

---

## Acceptance

- [ ] `relocate_service` RDP → 43389: TermService restarts on 43389, RDP connects on new port
- [ ] Rollback: if target port bind fails, original port restored within 10s
- [ ] Dashboard shows relocated state; bait Start enabled on classic port
- [ ] Port save validates availability against open_ports
- [ ] Firewall rule AR-RELOCATE-RDP-43389 created before restart
- [ ] Cloud: `settings_json.service_ports` persisted after success
- [ ] Client ≥ 4.9.44 handles `relocate_service` command type
