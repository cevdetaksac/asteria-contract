# Product branding — Asteria (`asteria.run`)

> **Contract VERSION:** **1.4.32**  
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
- [ ] Legacy host decommission (after fleet migrate)

## Client

Min **≥ 4.9.35** for signing cutover — full list in [`../agent/rebrand-asteria.md`](../agent/rebrand-asteria.md).
