# Cloud surface inventory (client gap-scan)

> **Contract:** ≥ **1.4.36** · **Audience:** Windows client developers  
> **Base:** `https://asteria.run` · **Control WS:** `wss://asteria.run/ws/agent/control`  
> **Purpose:** Single scannable list of what the cloud exposes to the agent — capabilities, HTTP paths, WS messages, command types.  
> **Detail SoT:** linked `api/*` / `agent/*` MDs remain normative for schemas; this file is the **index**.

### How the client team should use this

1. Filter rows with **Audience = agent** and **Status = live**.
2. Compare against your implemented endpoints / handlers / `command_type` switch.
3. Any row with `Min client ≤ your build` that you do not implement = **gap**.
4. Do **not** invent endpoints not listed here. Prefer fail-closed skip + alert.
5. Re-pull on each contract VERSION bump (`GET /api/public/contract`).

**Auth legend**

| Tag | Meaning |
|-----|---------|
| `Bearer` | `Authorization: Bearer <agent-token>` (JSON body `token` also accepted on some POSTs) |
| `WS-Bearer` | Control/remote WS: header or `Sec-WebSocket-Protocol: bearer,<token>` — **query `?token=` forbidden** on control WS |
| `Dash` | Browser dashboard / account cookie — agent must not call |
| `Public` | No agent token |

**Status legend:** `live` · `observe` (optional / flag-off) · `dash-only` · `deprecated`

---

## 0. Communication model (read this first)

```
┌─────────────────────────────┐
│ Windows SYSTEM daemon       │
│ (+ GUI via local IPC only)  │
└─────────────┬───────────────┘
              │ HTTPS Bearer
              │ WSS control  (/ws/agent/control)
              │ WSS remote   (/ws/remote/agent)  [RD only]
              ▼
         asteria.run (FastAPI)
```

| Channel | Use |
|---------|-----|
| HTTPS | register, heartbeat, attack, health, alerts, threat-intel, blocks poll, commands fallback, presence HTTP |
| Control WSS | **primary** command push + presence + intel/config invalidate hints |
| Remote WSS | Remote-desktop frames / signaling only — not IR commands |
| Browser `?token=` | Dashboard deep-links only — [`dashboard-deep-links.md`](./dashboard-deep-links.md) |

Signing: command HMAC context **`asteria-chp-v1`** · heartbeat **`asteria-heartbeat-v1`**  
([`../agent/rebrand-asteria.md`](../agent/rebrand-asteria.md), client ≥ **4.9.35**).

---

## 1. Agent HTTP — lifecycle & identity

| ID | Method | Path | Auth | Min client | Status | Notes / detail |
|----|--------|------|------|------------|--------|----------------|
| C-API-register | POST | `/api/register` | Public→returns token | — | live | First install; returns `dashboard`, `protection.block_rules` — [`../api/01-auth.md`](../api/01-auth.md) |
| C-API-rotate-token | POST | `/api/agent/rotate-token` | Bearer (old) | ≥4.9.31 | live | In-place token; same `client_id` — [`../api/01-auth.md`](../api/01-auth.md) |
| C-API-heartbeat | POST | `/api/heartbeat` | Bearer | — | live | Presence pulse; signed body context `asteria-heartbeat-v1` |
| C-API-update-ip | POST | `/api/update-ip` | Bearer | — | live | IP change; may archive superseded peers |
| C-API-presence | POST | `/api/presence` | Bearer | ≥4.9.8 | live | Sleep/suspend/shutdown HTTP fallback — [`../api/11-presence-realtime.md`](../api/11-presence-realtime.md) |
| C-API-account-status | GET | `/api/agent/account-status` | Bearer | ≥4.9.26 | live | `account_linked` — required for anti-brick — [`../api/02-account.md`](../api/02-account.md) |
| C-API-link-account | POST | `/api/agent/link-account` | Bearer | ≥4.9.26 | live | Optional direct link |
| C-API-unlink-account | POST | `/api/agent/unlink-account` | Bearer | ≥4.9.26 | live | Settings unlink |
| C-API-linked-servers | GET | `/api/agent/linked-servers` | Bearer | — | live | Same-account fleet (no cookie) |

---

## 2. Agent HTTP — attacks, health, events

| ID | Method | Path | Auth | Min client | Status | Notes |
|----|--------|------|------|------------|--------|-------|
| C-API-attack | POST | `/api/attack` | Bearer | — | live | Canonical attack ingest |
| C-API-honeypot-attack | POST | `/api/honeypot-attack` | Bearer | — | live | Alias |
| C-API-attacks-legacy | POST | `/attacks/` | Bearer | — | deprecated | Prefer `/api/attack` |
| C-API-health-report | POST | `/api/health/report` | Bearer | — | live | SystemHealth + settings mirror; cloud keeps ≤96 rows/client |
| C-API-health-latest | GET | `/api/health/latest` | Bearer | — | live | Latest snapshot |
| C-API-events-batch | POST | `/api/events/batch` | Bearer | — | live | Security events batch |
| C-API-open-ports | POST | `/api/agent/open-ports` | Bearer | — | live | Listening ports report |
| C-API-tunnel-status | POST | `/api/agent/tunnel-status` | Bearer | — | live | Agent → cloud tunnel state |
| C-API-tunnel-status-get | GET | `/api/premium/tunnel-status` | Bearer | — | live | Pending tunnel commands |
| C-API-tunnel-set | POST | `/api/premium/tunnel-set` | Bearer/Dash | — | live | Queue tunnel_start/stop |
| C-API-relocate-service | POST | `/api/premium/relocate-service` | Bearer/Dash | — | live | Queue relocate_service |
| C-API-relocate-port-save | POST | `/api/premium/relocate-port-save` | Bearer/Dash | — | live | Save preferred target port |
| C-API-relocate-report | POST | `/api/agent/relocate-report` | Bearer | — | live | GUI/local relocate → cloud sync |

---

## 3. Agent HTTP — threat engine & intel

| ID | Method | Path | Auth | Min client | Status | Notes |
|----|--------|------|------|------------|--------|-------|
| C-API-alerts-urgent | POST | `/api/alerts/urgent` | Bearer | — | live | Primary threat alert SoT — [`../agent/threat-engine.md`](../agent/threat-engine.md) |
| C-API-alerts-urgent-batch | POST | `/api/alerts/urgent/batch` | Bearer | observe | observe | Offline urgent queue — [`../api/10-offline-urgent-queue.md`](../api/10-offline-urgent-queue.md) |
| C-API-alerts-auto-block | POST | `/api/alerts/auto-block` | Bearer | — | live | Alias also `/api/v4/auto-block` |
| C-API-alerts-lifecycle | POST/GET | `/api/alerts/lifecycle` | Bearer | — | live | Lifecycle events |
| C-API-alerts-list | GET | `/api/alerts/list` | Bearer | — | live | |
| C-API-logon-challenge | POST | `/api/alerts/logon-challenge` | Bearer | — | live | Challenge create |
| C-API-logon-challenges | GET | `/api/agent/logon-challenges` | Bearer | — | live | Pending challenges for agent |
| C-API-threats-config | GET | `/api/threats/config` | Bearer | — | live | Full threat + network_guard + silent_hours + **`fleet_rollout`** (1.4.37) |
| C-API-threats-config-post | POST | `/api/threats/config` | Bearer/Dash | — | live | Update (prefer dashboard) |
| C-API-threats-summary | GET | `/api/threats/summary` | Bearer | — | live | |
| C-API-auto-blocks | GET | `/api/auto-blocks` | Bearer | — | live | |
| C-API-notif-prefs | PUT | `/api/notifications/preferences` | Bearer | — | live | |
| C-API-threat-intel | GET | `/api/agent/threat-intel` | Bearer | ≥4.5.61 | live | ETag bundle — [`../api/09-threat-intel.md`](../api/09-threat-intel.md) |
| C-API-threat-intel-ack | POST | `/api/agent/threat-intel/ack` | Bearer | ≥4.5.61 | live | |
| C-API-security-resilience | GET | `/api/security-resilience/status` | Bearer | observe | observe | P1 observe schemas |

---

## 4. Agent HTTP — firewall / blocks

| ID | Method | Path | Auth | Min client | Status | Notes |
|----|--------|------|------|------------|--------|-------|
| C-API-pending-blocks | GET | `/api/agent/pending-blocks` | Bearer | — | live | Also pulses presence |
| C-API-pending-unblocks | GET | `/api/agent/pending-unblocks` | Bearer | — | live | Whitelist lift / unblock queue |
| C-API-block-applied | POST | `/api/agent/block-applied` | Bearer | — | live | ACK applied |
| C-API-block-removed | POST | `/api/agent/block-removed` | Bearer | — | live | ACK removed |
| C-API-sync-rules | POST | `/api/agent/sync-rules` | Bearer | — | live | Snapshot/replace AR-BLOCK inventory — [`../api/06-firewall-blocks.md`](../api/06-firewall-blocks.md) |
| C-API-block-rules-list | GET | `/api/premium/block-rules` | Bearer | — | live | |
| C-API-operator-keys | GET | `/api/agent/operator-keys` | Bearer | design | observe | ZT keys — design-only until promoted |
| C-API-clear-data | POST | `/api/agent/clear-data` | Bearer | — | live | Queues clear_data / clear_firewall |

Brand prefixes: **`AR-BLOCK-*` / `AR-INTEL-*`** (client ≥4.9.33).

---

## 5. Agent HTTP — commands (HTTP fallback)

Primary delivery = Control WS `t:command`. HTTP remains mandatory fallback.

| ID | Method | Path | Auth | Status | Notes |
|----|--------|------|------|--------|-------|
| C-API-commands-pending | GET | `/api/commands/pending` | Bearer | live | Poll fallback; pulses presence |
| C-API-commands-result | POST | `/api/commands/result` | Bearer | live | Status = command state (`completed`…), **not** SAM `active`/`disabled` |
| C-API-commands-get | GET | `/api/commands/{command_id}` | Bearer | live | |
| C-API-commands-send | POST | `/api/commands/send` | Bearer/Dash | live | Queue; destructive needs `confirm:true` |
| C-API-commands-cancel | DELETE | `/api/commands/{command_id}` | Bearer/Dash | live | |

Catalog whitelist = `helpers.VALID_COMMAND_TYPES` (§7). Detail: [`../api/03-control-websocket.md`](../api/03-control-websocket.md).

---

## 6. Control WebSocket

| ID | Direction | `t` | Min client | Status | Client must |
|----|-----------|-----|------------|--------|-------------|
| C-WS-connect | — | `wss://…/ws/agent/control` | — | live | Connect as daemon; Bearer only |
| C-WS-hello | agent→cloud | `hello` | — | live | Send on connect (`version`, `mode=daemon`, optional `caps`) |
| C-WS-hello-ack | cloud→agent | `hello_ack` | — | live | Treat as online; expect pending drain |
| C-WS-ping | either | `ping` / `pong` | — | live | ~25–30s keepalive |
| C-WS-command | cloud→agent | `command` | — | live | Verify HMAC `asteria-chp-v1`; execute; HTTP result |
| C-WS-command-ack | agent→cloud | `command_ack` | — | live | Optional short ack |
| C-WS-intel | cloud→agent | `threat_intel_updated` | ≥4.5.61 | live | Immediate `GET /api/agent/threat-intel` |
| C-WS-config | cloud→agent | `threat_config_updated` | ≥4.7.3 | live | Immediate `GET /api/threats/config` + apply |
| C-WS-unblocks | cloud→agent | `pending_unblocks_updated` | ≥4.9.x / 1.4.11 | live | Immediate `GET /api/agent/pending-unblocks` |
| C-WS-presence | agent→cloud | `presence` | ≥4.9.8 | live | suspend/offline reasons |
| C-WS-goodbye | agent→cloud | `goodbye` | ≥4.9.8 | live | shutdown path |
| C-WS-presence-ack | cloud→agent | `presence_ack` | ≥4.9.8 | live | |
| C-WS-error | cloud→agent | `error` | — | live | Log + reconnect policy |

Remote desktop WS (separate): `/ws/remote/agent`, `/ws/remote/view` — [`../api/05-remote-desktop.md`](../api/05-remote-desktop.md).

---

## 7. Command catalog (cloud whitelist)

Cloud refuses unknown `command_type`. Source: `helpers.VALID_COMMAND_TYPES`.  
**D** = destructive (`confirm:true` required unless noted).

### Firewall / process / account

| command_type | D | Min client | Detail |
|--------------|---|------------|--------|
| `block_ip` | | — | |
| `unblock_ip` | | — | |
| `clear_firewall` | D* | — | *D if `wipe_all_honeypot_rules` |
| `sync_firewall_rules` | | — | |
| `list_firewall` | | ≥**4.9.41** | All inbound/outbound + profiles (`scope=all`); 4.9.40 Asteria-only degraded |
| `firewall_set_profile` | D | ≥**4.9.40** | Confirm-gated profile mutate |
| `firewall_rule` | D* | ≥**4.9.41** | `enable`/`disable`/`delete`/`add` — *D for delete/add |
| `logoff_user` | | — | |
| `disable_account` | D | — | Anti-brick gated locally — [`../agent/anti-brick-critical-actions.md`](../agent/anti-brick-critical-actions.md) |
| `enable_account` | | — | |
| `disable_all_users` | D | — | |
| `reset_password` | D | — | |
| `kill_process` | D | — | |
| `suspend_process` | D | ≥4.7.3 | |
| `resume_process` | | ≥4.7.3 | |
| `allow_process` | D | ≥4.7.3 | |
| `block_process` | | — | |
| `stop_service` / `start_service` / `restart_service` | | ≥4.9.4 | |
| `enable_lockdown` / `disable_lockdown` | D / — | — | |
| `list_sessions` / `list_local_users` / `list_processes` | | ≥4.9.4 | |
| `collect_diagnostics` | | — | |

### Remote desktop / tunnels / update

| command_type | D | Min client | Detail |
|--------------|---|------------|--------|
| `remote_stream_start` / `remote_stream_stop` | | ≥4.9.0 | |
| `remote_input` / `remote_send_sas` | | ≥4.9.0 | |
| `remote_session_prepare` | | ≥4.9.0 | password scrubbed in audit |
| `tunnel_start` / `tunnel_stop` | | — | |
| `relocate_service` | D | ≥**4.9.45** | Real-service port move + golden rollback; report via result / relocate-report |
| `self_update` / `check_update` | | ≥4.5.39 | |
| `unlock_ransomware_quarantine` | | ≥4.5.65 | |

### Disaster recovery / Network Guard / System Recovery / GUI PIN

| command_type | D | Min client | Detail |
|--------------|---|------------|--------|
| `create_user` / `remote_logon` | D | ≥4.6.0 | |
| `set_autologon` / `clear_autologon` | D / — | ≥4.6.0 | |
| `reboot` | D | ≥4.6.0 | |
| `network_snapshot` / `list_network_baseline` / `network_diff` | | ≥4.7.0+ | |
| `network_restore` | D† | ≥4.7.0 | †exempt if `dry_run:true` |
| `network_accept_surface` | | ≥4.9.15 | |
| `network_disable_adapter` | D† | ≥4.9.15 | †exempt if `dry_run:true` |
| `network_adapter_apply` | D | ≥**4.9.42** | Enable/IP/DNS + local golden watchdog |
| `network_maintenance_start` / `network_maintenance_end` | | ≥4.9.12 | |
| `system_recovery_snapshot` / `list_system_recovery` / `system_recovery_diff` | | ≥4.9.12 | |
| `system_recovery_restore` | D† | ≥4.9.12 | †exempt if `dry_run:true` |
| `set_gui_pin` / `clear_gui_pin` | D | ≥4.8.3 | `pin` scrubbed |

**Not in whitelist (do not send):** `contain_user` — compose `logoff_user` + `reset_password` (+ optional `disable_account`).

---

## 8. Remote desktop HTTP (agent + viewer)

| ID | Method | Path | Audience | Status |
|----|--------|------|----------|--------|
| C-RD-frame | POST | `/api/remote/frame` | agent | live |
| C-RD-frame-json | POST | `/api/remote/frame-json` | agent | live |
| C-RD-status | GET | `/api/remote/status` | dash/agent | live |
| C-RD-frame-jpg | GET | `/api/remote/frame.jpg` | dash | live |
| C-RD-input | POST | `/api/remote/input` | dash | live |
| C-RD-inputs | GET | `/api/remote/inputs` | agent | live |
| C-RD-session | POST | `/api/remote/session` | dash | live |
| C-RD-users | GET | `/api/remote/users` | dash | live |
| C-RD-users-refresh | POST | `/api/remote/users/refresh` | dash | live |
| C-RD-prepare | POST | `/api/remote/prepare` | dash | live |
| C-RD-cad | POST | `/api/remote/cad` | dash | live |
| C-RD-ws-agent | WS | `/ws/remote/agent` | agent | live |
| C-RD-ws-view | WS | `/ws/remote/view` | dash | live |

---

## 9. Dashboard deep-links (browser only)

Not agent HTTP. Token in query is intentional for operators/GUI “Open dashboard”.

Canonical list: [`dashboard-deep-links.md`](./dashboard-deep-links.md)

Pattern: `https://asteria.run{path}?token={TOKEN}`

---

## 10. Explicitly out of agent scope (do not implement as agent API)

| Area | Examples |
|------|----------|
| Marketing / site | `/`, `/about`, `/pricing`, `/download`, … |
| Account HTML | `/servers`, `/account/login`, `/account/link-server`, … |
| Dashboard HTML | `/dashboard`, `/dashboard/blocks`, … |
| Dashboard JSON | `/api/dashboard-stats`, `/api/dashboard-live`, `/api/dashboard/*`, `/api/client_status` |
| Billing | `/billing/*` |
| Public meta | `/api/public/contract`, `/api/public/latest-release`, `/healthz` |
| Threat-intel admin | `/api/threat-intel/publish`, allowlist CRUD (cloud ops) |

Agent may **open** dashboard URLs in the OS browser; it must not scrape dashboard HTML as an API.

---

## 11. Client gap-scan checklist (copy into PR)

```
Contract VERSION: ______
Client build: ______

[ ] §1 lifecycle (register/heartbeat/rotate/presence/account-status)
[ ] §2 attack + health/report cadence
[ ] §3 alerts/urgent + threats/config + threat-intel GET/ACK
[ ] §4 pending-blocks / pending-unblocks / sync-rules (AR-BLOCK)
[ ] §5 commands/pending + commands/result (status vocabulary)
[ ] §6 control WS: hello, ping, command, threat_intel_updated, threat_config_updated,
      pending_unblocks_updated, presence/goodbye
[ ] §7 command_types for features you claim (list gaps: ________)
[ ] Signing: asteria-chp-v1 + asteria-heartbeat-v1 (≥4.9.35)
[ ] Anti-brick: skip critical auto if !account_linked (≥4.9.36)
[ ] No ?token= on agent HTTP/WS
```

---

## 12. Maintenance

- Adding a cloud agent endpoint or `VALID_COMMAND_TYPES` entry → update **this file** in the same contract VERSION bump.
- Cloud code SoT paths: `routes_agent.py`, `routes_v4.py`, `routes_control.py`, `routes_threat_intel.py`, `routes_remote.py`, `helpers.VALID_COMMAND_TYPES`.
- Human architecture sketch: [`overview.md`](./overview.md).
