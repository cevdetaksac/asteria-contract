# Product branding — Asteria (`asteria.run`)

> **Contract VERSION:** **1.4.37** (sunset plan added; cutover wire still **1.4.32**)  
> **Normative brand migration SoT:** [`../agent/rebrand-asteria.md`](../agent/rebrand-asteria.md)  
> Do not duplicate legacy name tables elsewhere — point here / to that file.

## Karar

| | |
|--|--|
| **Brand** | **Asteria** |
| **Primary domain** | **`https://asteria.run`** |
| **Legacy domain** | `https://honeypot.yesnext.com.tr` (nginx alias until fleet migrates) |
| **Tagline** | Deception Cloud |
| **Former name** | YesNext Cloud Honeypot (historical only — see rebrand SoT) |

## Wire (current)

| Katman | Değer |
|--------|--------|
| Firewall | **`AR-BLOCK-*` / `AR-INTEL-*`** (wipe still clears `HP-*` / `HONEYPOT*`) |
| Command signing | **`asteria-chp-v1`** (cloud signs); verify also accepts legacy `yesnext-chp-v1` during cutover |
| Heartbeat signing | **`asteria-heartbeat-v1`** |
| Client releases | `cevdetaksac/asteria-client` |
| API paths | unchanged (`/api/...`, `/ws/...`) |

## Cloud checklist (1.4.32)

- [x] nginx `asteria.run` + www + legacy alias
- [x] Deploy `/data/asteria.run`, PM2 `asteria-api`
- [x] `brand.py` + `PUBLIC_BASE_URL`
- [x] UI / CSS `--ast-*` / OpenAPI / mail Asteria
- [x] Command HMAC `asteria-chp-v1`
- [x] `SECRET_KEY` / `LOGON_CHALLENGE_SECRET` required (no hardcoded defaults)
- [ ] Legacy host decommission (after fleet migrate) — see **Sunset** below

## Sunset — dual-brand / legacy host (normative plan)

| | |
|--|--|
| **Target date** | **2026-10-01** (UTC) |
| **Goal** | Remove nginx dependency on `honeypot.yesnext.com.tr` as supported API alias; stop documenting legacy HMAC verify as required cutover path |

### Exit criteria (ALL required before decommission)

1. Fleet report: **≥99%** of heartbeating agents in last 14d use primary host `asteria.run` (not legacy alias as sole reachability).  
2. Fleet report: **≥99%** of agents verify **`asteria-chp-v1`** (no production hosts stuck on `yesnext-chp-v1`-only verify).  
3. Client floor for linked production accounts ≥ **4.9.35** (signing) / recommended ≥ **4.9.37** (canary + RD P0).  
4. Ops runbook: emergency re-enable legacy alias ≤15 minutes (DNS/nginx) for 30 days after cut.  
5. Contract VERSION explicitly marks legacy host **deprecated** then a later VERSION **removed**.

### Interim (until 2026-10-01)

- Legacy alias **stays** for failover.  
- Cloud **signs** only `asteria-chp-v1`.  
- Wipe lists keep `HP-*` / `HONEYPOT*`.  
- ProgramData silent-move still forbidden.

If criteria miss the date: publish a contract note slipping the date — do **not**
silent-cut the alias.

## Visual identity (UI)

| Token | HEX | Use |
|-------|-----|-----|
| Steel-navy | `#3f4d5e` | Tower gradient top |
| Anthracite | `#080d14` | Background / tower bottom |
| Cyan | `#2ea8d1` | ASTERIA wordmark, sparkles, primary CTA |
| Steel-grey | `#8792a0` | RUN wordmark, muted UI |

- **Brand font:** [Bruno Ace](https://fonts.google.com/specimen/Bruno+Ace) Regular — uppercase **ASTERIA** with wide letter-spacing; body UI stays Figtree.
- **Design source:** deploy root `logo_set/` (full-res PNGs).
- **Web assets:** `/static/brand/` — regenerated via `scripts/export_brand_assets.py`.
  - `logo-horizontal-light*.png` — **dark UI** (bright cyan ink)
  - `logo-horizontal*.png` (no `-light`) — **light UI** (steel-filled mark)
  - Same rule for `logo-stacked*` / `mark*` / favicon
  - `icon-{online,offline,disabled,stay}*.png` — agent status
- Theme switch: dashboard `data-bs-theme` + `prefers-color-scheme` for marketing/favicon.
- Product descriptor **Deception Cloud** remains copy; logo lockup is **ASTERIA RUN**.

## Client

Min **≥ 4.9.35** for signing cutover — full list in [`../agent/rebrand-asteria.md`](../agent/rebrand-asteria.md).
