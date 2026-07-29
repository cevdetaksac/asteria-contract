# Firewall Management — Asteria inventory (1.4.40)

> **Contract VERSION:** **1.4.40** (superseded for full host parity by **1.4.41**  
> [`firewall-windows-parity.md`](./firewall-windows-parity.md) — client ≥ **4.9.41**)  
> Status: **Normative for Asteria-scoped inventory (client ≥ 4.9.40)**; keep implementing  
> until clients upgrade to full `scope=all` listing.  
> Related: [`../api/06-firewall-blocks.md`](../api/06-firewall-blocks.md) ·
> [`firewall-brand-migrate.md`](./firewall-brand-migrate.md) ·
> [`system-recovery.md`](./system-recovery.md) (profile snapshot subset)

## Goal

Operators need one page to manage **Asteria firewall rules** and inspect the
**host Windows Firewall** state (profiles + Asteria-prefixed rules), without
confusing that with attack-history counts.

**1.4.41+** expands this to full Windows Firewall MMC parity (all inbound/outbound
rules + enable/disable/delete/add). See
[`firewall-windows-parity.md`](./firewall-windows-parity.md).

Cloud page: `GET /dashboard/server/firewall?token=`  
Deep-link: `https://asteria.run/dashboard/server/firewall?token={TOKEN}`

---

## Existing (keep)

| Piece | Notes |
|-------|--------|
| `block_ip` / `unblock_ip` | Dashboard + IR |
| `sync_firewall_rules` | Agent → `POST /api/agent/sync-rules` (AR-BLOCK inventory → cloud) |
| `clear_firewall` | Wipe Asteria/legacy prefixes |
| `POST /api/premium/request-firewall-sync` | Queue `sync_firewall_rules` |
| Blocking page | Thresholds / whitelist / seed rules — **not** replaced |

---

## Commands (client ≥ 4.9.40 — Asteria scope)

### 1) `list_firewall` (read-only, high)

Params (all optional):

```json
{
  "include_profiles": true,
  "include_asteria_rules": true,
  "include_counts": true,
  "include_non_asteria_summary": true,
  "max_rules": 500
}
```

> **1.4.41:** Prefer `scope=all` + `include_inbound` / `include_outbound` +
> `max_rules_per_direction` per [`firewall-windows-parity.md`](./firewall-windows-parity.md).
> Asteria-only responses remain valid as **degraded**.

Result `data`:

```json
{
  "profiles": {
    "domain":  { "state": "on", "inbound": "block", "outbound": "allow" },
    "private": { "state": "on", "inbound": "block", "outbound": "allow" },
    "public":  { "state": "on", "inbound": "block", "outbound": "allow" }
  },
  "asteria_rules": [
    {
      "name": "AR-BLOCK-203.0.113.10",
      "direction": "In",
      "action": "Block",
      "enabled": true,
      "remote_address": "203.0.113.10",
      "protocol": "Any",
      "prefix": "AR-BLOCK"
    }
  ],
  "counts": {
    "asteria_rules": 42,
    "ar_block": 40,
    "ar_intel": 2,
    "hp_legacy": 0,
    "inbound_block": 120,
    "total_rules": 380
  },
  "captured_at": "2026-07-29T12:00:00Z"
}
```

Rules:

| ID | Rule |
|----|------|
| **C-FW-LIST-1** | Enumerate **enabled + disabled** inbound rules whose name starts with `AR-BLOCK`, `AR-INTEL`, `HP-BLOCK`, `HP-INTEL`, `HONEYPOT` (case-insensitive) |
| **C-FW-LIST-2** | Always return `profiles` for Domain / Private / Public (`state`, default inbound/outbound) |
| **C-FW-LIST-3** | Cap `asteria_rules` at `max_rules` (default 500); put totals in `counts` even when truncated |
| **C-FW-LIST-4** | 1.4.40: do **not** dump every Windows rule by default — non-Asteria only as **counts**. **1.4.41** opts in via `scope=all` |
| **C-FW-LIST-5** | On success, cloud caches `data` in `client.settings.firewall_inventory_cache` for the dashboard |

Also continue pushing IP list via existing `sync_firewall_rules` / `sync-rules` so Applied Blocks stays accurate.

### 2) `firewall_set_profile` (destructive confirm)

Params:

```json
{
  "profile": "public",
  "state": "on",
  "inbound": "block",
  "outbound": "allow"
}
```

| ID | Rule |
|----|------|
| **C-FW-PROF-1** | Require cloud `confirm:true` (destructive) |
| **C-FW-PROF-2** | Only mutate the named profile; refuse unknown profile names |
| **C-FW-PROF-3** | After change, emit a fresh `list_firewall` result (or operator re-lists) |

---

## Cloud (shipped ≥1.4.40, parity UI ≥1.4.41)

- Nav: Sunucu Yönetimi → **Firewall Yönetimi**
- Page: Profiles / Inbound / Outbound / Asteria tabs (≥1.4.41)
- `GET /api/firewall/overview` — soft refresh JSON
- Command result hook caches `list_firewall` → `firewall_inventory_cache`
- Page-enter sync calls `list_firewall` + `sync_firewall_rules` quietly when agent online

---

## Acceptance (1.4.40 floor)

- [ ] Client ≥4.9.40: profiles + Asteria rule rows within 15s  
- [ ] Sync button still updates Applied Blocks via `sync-rules`  
- [ ] `clear_firewall` / unblock still work from this page  
- [ ] Upgrade to ≥4.9.41 for full MMC parity acceptance in `firewall-windows-parity.md`
