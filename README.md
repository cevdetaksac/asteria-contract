# Asteria — Shared Contract

**Single source of truth** for Windows client ↔ Cloud API.

| | |
|--|--|
| **VERSION** | [`VERSION`](VERSION) (**1.4.70**) |
| **Features** | [`features/README.md`](features/README.md) — one MD per product |
| **Cloud checklist** | [`cloud/CLOUD_CHECKLIST.md`](cloud/CLOUD_CHECKLIST.md) — 1.4.70 lock-follow items open |
| **Client checklist** | [`features/CLIENT_CHECKLIST.md`](features/CLIENT_CHECKLIST.md) — lab host ticks |
| **Index** | [`INDEX.md`](INDEX.md) |
| **Fleet** | [`FLEET.md`](FLEET.md) — floor client ≥ **4.9.0** |
| **API** | `https://asteria.run` · Bearer token (not `?token=` on agent API) |
| **Signing** | HMAC **`asteria-chp-v1`** · heartbeat **`asteria-heartbeat-v1`** |

## Who reads what

| Party | Start |
|-------|--------|
| **Cloud / dashboard** | [`cloud/CLOUD_CHECKLIST.md`](cloud/CLOUD_CHECKLIST.md) then the linked `features/*` |
| **Windows client** | [`features/CLIENT_CHECKLIST.md`](features/CLIENT_CHECKLIST.md) then the linked `features/*` |
| **Cursor / agents** | `VERSION` → `features/README.md` → product MD |

## Rules

1. Behavior change → MD + CHANGELOG + VERSION here **first**, then code.
2. Feature file wins over `agent/` `api/` `cloud/` appendices.
3. Client does not pull external threat feeds.
4. No PM2 / nginx / dashboard HTML in this repo.

## Clone / publish

```bash
git clone https://github.com/cevdetaksac/asteria-contract.git
cat VERSION   # expect 1.4.68+
```

Cloud host: `cd /data/asteria.run/contract && git pull && ../scripts/publish_contract.sh`  
Mirror: `https://asteria.run/static/shared-contract.zip`
