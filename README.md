# Asteria — Shared Contract

**Single source of truth** for Windows client ↔ Cloud API behavior.

| | |
|--|--|
| **VERSION** | [`VERSION`](VERSION) (**1.4.33**) |
| **Index** | [`INDEX.md`](INDEX.md) |
| **Changelog** | [`CHANGELOG.md`](CHANGELOG.md) |
| **Fleet matrix** | [`FLEET.md`](FLEET.md) — production floor client ≥ **4.9.0** |
| **Brand SoT** | [`agent/rebrand-asteria.md`](agent/rebrand-asteria.md) · [`cloud/PRODUCT_BRANDING.md`](cloud/PRODUCT_BRANDING.md) |
| **API base** | `https://asteria.run` (legacy nginx alias until fleet migrates — details in rebrand SoT) |
| **Auth** | `Authorization: Bearer <token>` — agent API must not rely on `?token=` query |
| **Signing** | Command HMAC **`asteria-chp-v1`** · heartbeat **`asteria-heartbeat-v1`** (client ≥ **4.9.35**) |

## Who uses this?

| Party | Role |
|-------|------|
| **Windows client** | Implements the contract; does not invent endpoints |
| **Cloud / API** | Implements the same MDs; PM2/nginx/dashboard HTML are **not** in this repo |
| **Cursor / agents** | Read `VERSION` → `INDEX.md` → relevant MD before coding |

## Client start here

1. [`VERSION`](VERSION) → [`INDEX.md`](INDEX.md) → [`agent/CLIENT.md`](agent/CLIENT.md)  
2. Cutover checklist: [`agent/rebrand-asteria.md`](agent/rebrand-asteria.md) (**1.4.32+**, client ≥ **4.9.35**)  
3. Firewall prefixes: [`agent/firewall-brand-migrate.md`](agent/firewall-brand-migrate.md)  
4. Min versions: [`FLEET.md`](FLEET.md)

## Rules

1. Behavior / API change → **first** MD + CHANGELOG + VERSION bump here → then code.
2. Ambiguity → add `## Open questions` in the MD; do not guess.
3. Client does not pull external threat feeds; only the cloud threat-intel bundle.
4. Minimum client version is stated per MD (see `FLEET.md`).
5. No cloud-only ops (PM2, nginx, dashboard HTML) in this repo.
6. Design-only / roadmap docs are marked as such — wire SoT is `api/` + `agent/`.
7. Former YesNext / Honeypot names live **only** in [`agent/rebrand-asteria.md`](agent/rebrand-asteria.md).

## Clone

```bash
git clone https://github.com/cevdetaksac/asteria-contract.git
cd asteria-contract
cat VERSION   # expect 1.4.33+
```

Client workspace pointer: `asteria-client/contract/` → this repo.

## Layout

| Path | Content |
|------|---------|
| `api/` | HTTP / WS wire contracts |
| `agent/` | Client-side behavior |
| `cloud/` | Cloud must-do (C-* checklists) + marked design/roadmap notes |

## Cloud publish (operators)

```bash
cd /data/asteria.run/contract && git pull && ../scripts/publish_contract.sh
```

HTTPS mirror: `https://asteria.run/static/shared-contract.zip`  
Meta: `GET /api/public/contract`
