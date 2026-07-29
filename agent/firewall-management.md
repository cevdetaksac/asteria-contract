# Firewall Management — full host inventory + dashboard page

> **Contract VERSION:** **1.4.40**  
> Status: **Normative (client ≥ 4.9.40)**  
> Related: [`../api/06-firewall-blocks.md`](../api/06-firewall-blocks.md) ·
> [`firewall-brand-migrate.md`](./firewall-brand-migrate.md) ·
> [`system-recovery.md`](./system-recovery.md) (profile snapshot subset)

## Goal

Operators need one page to manage **Asteria firewall rules** and inspect the
**host Windows Firewall** state (profiles + Asteria-prefixed rules), without
confusing that with attack-history counts.

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

## New commands (client MUST)

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
| **C-FW-LIST-4** | Do **not** dump every Windows rule by default — non-Asteria only as **counts** (`inbound_block`, `total_rules`) unless a future param opts in |
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

Dashboard may expose this later; cloud accepts the command type now.

---

## Cloud (shipped ≥1.4.40)

- Nav: Sunucu Yönetimi → **Firewall Yönetimi** (under Port Yönetimi)
- Page: cloud mirror tables + profiles + inventory from cache
- `GET /api/firewall/overview` — soft refresh JSON
- Command result hook caches `list_firewall` → `firewall_inventory_cache`
- Page-enter sync calls `list_firewall` + `sync_firewall_rules` quietly when agent online

---

## Acceptance

- [ ] Client ≥4.9.40: dashboard **Tam envanter** → profiles + Asteria rule rows within 15s  
- [ ] Sync button still updates Applied Blocks via `sync-rules`  
- [ ] `clear_firewall` / unblock still work from this page  
- [ ] Old clients (<4.9.40): page still useful (cloud mirror + sync); inventory shows empty + hint  
