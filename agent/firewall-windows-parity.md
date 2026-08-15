# Firewall Management — Windows Firewall parity

> **Behavior SoT:** [`../features/firewall.md`](../features/firewall.md) (**≥ 1.4.62**). Appendix below.

> **Contract VERSION:** **1.4.41**  
> Status: **Normative (client ≥ 4.9.41)**  
> Related: [`../api/06-firewall-blocks.md`](../api/06-firewall-blocks.md) ·
> [`firewall-brand-migrate.md`](./firewall-brand-migrate.md) ·
> [`firewall-management.md`](./firewall-management.md) *(1.4.40 — superseded by this doc for host parity)* ·
> [`system-recovery.md`](./system-recovery.md)

## Goal

The dashboard **Firewall Yönetimi** page MUST be able to show and drive the same
operator actions as the Windows **Windows Defender Firewall with Advanced
Security** MMC for day-to-day IR:

| Windows UI surface | Dashboard |
|--------------------|-----------|
| Profile on/off (Domain / Private / Public) | Profiles tab |
| Default inbound / outbound action per profile | Profiles tab |
| Inbound rules list (all) | Inbound tab |
| Outbound rules list (all) | Outbound tab |
| Enable / Disable rule | Row actions |
| Delete rule | Row actions (confirm) |
| New rule (program / port / IP / custom subset) | New rule form |
| Asteria `AR-*` / legacy `HP-*` IP blocks | Asteria tab + existing `block_ip` |

**Out of scope (v1):** Connection Security (IPsec) rules, GPO-locked rule edit
when Windows returns access denied (surface error honestly), advanced
edge-traversal / interface-type rare flags (forward-compat ignore).

Cloud page: `GET /dashboard/server/firewall?token=`  
Deep-link: `https://asteria.run/dashboard/server/firewall?token={TOKEN}`

---

## Existing Asteria block pipeline (keep)

| Piece | Notes |
|-------|--------|
| `block_ip` / `unblock_ip` | Fast IR IP block → `AR-BLOCK-*` |
| `sync_firewall_rules` + `POST /api/agent/sync-rules` | Cloud Applied Blocks SoT |
| `clear_firewall` | Wipe Asteria/legacy prefixes only |
| Blocking page | Thresholds / whitelist — not replaced |

---

## Commands (client MUST ≥ 4.9.41)

### 1) `list_firewall` (read-only)

Params:

```json
{
  "include_profiles": true,
  "include_inbound": true,
  "include_outbound": true,
  "include_asteria_rules": true,
  "include_counts": true,
  "scope": "all",
  "max_rules_per_direction": 2000,
  "offset_in": 0,
  "offset_out": 0
}
```

- `scope`: `all` | `asteria` (name prefix filter only)
- Caps apply **per direction**; always return honest `counts` + `truncated_*`

Result `data`:

```json
{
  "profiles": {
    "domain":  { "state": "on", "inbound": "block", "outbound": "allow" },
    "private": { "state": "on", "inbound": "block", "outbound": "allow" },
    "public":  { "state": "on", "inbound": "block", "outbound": "allow" }
  },
  "inbound_rules": [ /* Rule */ ],
  "outbound_rules": [ /* Rule */ ],
  "asteria_rules": [ /* subset convenience; may duplicate inbound/outbound */ ],
  "counts": {
    "inbound_total": 812,
    "outbound_total": 640,
    "inbound_enabled": 700,
    "outbound_enabled": 600,
    "asteria_rules": 42,
    "ar_block": 40,
    "ar_intel": 2,
    "hp_legacy": 0
  },
  "truncated_inbound": false,
  "truncated_outbound": false,
  "captured_at": "2026-07-29T14:00:00Z",
  "engine": "netsh|com|powershell"
}
```

**Rule object (canonical):**

```json
{
  "name": "Remote Desktop - User Mode (TCP-In)",
  "group": "Remote Desktop",
  "enabled": true,
  "direction": "In",
  "action": "Allow",
  "profile": "Domain,Private,Public",
  "program": "%SystemRoot%\\system32\\svchost.exe",
  "local_address": "Any",
  "remote_address": "Any",
  "protocol": "TCP",
  "local_port": "3389",
  "remote_port": "Any",
  "edge_traversal": "No",
  "grouping": "…",
  "description": "…",
  "asteria_prefix": null
}
```

| ID | Rule |
|----|------|
| **C-FW-ALL-1** | With `scope=all`, return **all** inbound and outbound rules (enabled + disabled), not only Asteria |
| **C-FW-ALL-2** | Profiles always included when `include_profiles` |
| **C-FW-ALL-3** | Cap lists; set `truncated_*` + full totals in `counts` |
| **C-FW-ALL-4** | Asteria-prefixed names set `asteria_prefix` (`AR-BLOCK` / `AR-INTEL` / `HP-*` / `HONEYPOT`) |
| **C-FW-ALL-5** | Cloud caches full `data` in `firewall_inventory_cache` (may store last successful list only) |

### 2) `firewall_set_profile` (destructive confirm)

```json
{
  "profile": "public",
  "state": "off",
  "inbound": "block",
  "outbound": "allow"
}
```

| ID | Rule |
|----|------|
| **C-FW-PROF-1** | Cloud `confirm:true` required |
| **C-FW-PROF-2** | `profile` ∈ `domain`\|`private`\|`public` (or `all` = apply to each) |
| **C-FW-PROF-3** | `state` ∈ `on`\|`off`; omit fields = leave unchanged |
| **C-FW-PROF-4** | After mutate, prefer returning updated `profiles` in result |

> Turning a profile **off** is host-wide IR impact — dashboard sticky confirm.

### 3) `firewall_rule` (mutate — confirm by `op`)

Unified rule op (preferred over many tiny command types):

```json
{
  "op": "disable",
  "name": "Remote Desktop - User Mode (TCP-In)",
  "direction": "In"
}
```

| `op` | Confirm | Body extras |
|------|---------|-------------|
| `enable` | no* | `name`, optional `direction` |
| `disable` | no* | same |
| `delete` | **yes** | same |
| `add` | **yes** | full New-rule fields below |

\*Enable/disable of **Asteria** rules may still require confirm if product policy says so; default = no confirm for enable/disable, confirm for delete/add.

**Add / upsert fields:**

```json
{
  "op": "add",
  "name": "AR-MANUAL-203.0.113.10",
  "direction": "In",
  "action": "Block",
  "enabled": true,
  "profile": "any",
  "protocol": "Any",
  "local_port": "Any",
  "remote_port": "Any",
  "local_address": "Any",
  "remote_address": "203.0.113.10",
  "program": "Any",
  "description": "Dashboard manual"
}
```

| ID | Rule |
|----|------|
| **C-FW-RULE-1** | Match rule by exact `name` (+ `direction` if ambiguous duplicates) |
| **C-FW-RULE-2** | `delete` / `add` require cloud `confirm:true` |
| **C-FW-RULE-3** | GPO-locked / access-denied → `success:false` + `error=ACCESS_DENIED` or `GPO_LOCKED` |
| **C-FW-RULE-4** | After successful mutate, return `{ name, op, enabled? }` and ideally refresh list snapshot |
| **C-FW-RULE-5** | Prefer creating Asteria-prefixed names for dashboard-originated IP blocks (`AR-MANUAL-*` or reuse `block_ip`) |

---

## Cloud dashboard (shipped ≥ 1.4.41)

Tabs:

1. **Profiles** — on/off + default inbound/outbound + Apply  
2. **Inbound** — searchable table, enable/disable/delete, New rule  
3. **Outbound** — same  
4. **Asteria** — cloud Applied / Pending + clear Asteria / sync  
5. Refresh = `list_firewall` (`scope=all`)

Page-enter soft sync queues `list_firewall` when agent online.

---

## Acceptance

- [ ] Lab: Profiles show same on/off as `wf.msc` within one refresh  
- [ ] Lab: Inbound list length ≈ Windows inbound count (± truncation banner)  
- [ ] Lab: Disable rule in dashboard → disabled in `wf.msc` ≤ 5s  
- [ ] Lab: Profile Public Off → host public firewall off  
- [ ] Lab: Delete confirm required; cancel leaves rule  
- [ ] Old client (<4.9.41): Profiles/Inbound empty + upgrade hint; Asteria tab still works  

---

## Min client

**≥ 4.9.41** (or next train that closes C-FW-ALL / C-FW-PROF / C-FW-RULE).  
1.4.40 `list_firewall` Asteria-only responses remain accepted as degraded until upgraded.
