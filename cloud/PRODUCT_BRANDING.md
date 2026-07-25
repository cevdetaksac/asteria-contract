# Product branding — Asteria (`asteria.run`)

> **Contract VERSION:** **1.4.30** (brand cutover)  
> Statü: **normative** — cloud live; client migration required ≥ **4.9.32**  
> Detay checklist: [`../agent/rebrand-asteria.md`](../agent/rebrand-asteria.md)

## Karar

| | |
|--|--|
| **Brand** | **Asteria** |
| **Primary domain** | **`https://asteria.run`** |
| **Legacy domain** | `https://honeypot.yesnext.com.tr` (alias; keep until fleet migrated) |
| **Tagline** | Deception Cloud |
| **Former name** | YesNext Cloud Honeypot |

Cloud dashboard, e-posta, public contract mirror ve register/rotate `dashboard` URL'leri
`PUBLIC_BASE_URL=https://asteria.run` üzerinden yayınlanır.

## Wire / OS kimlikleri

| Katman | Karar | Not |
|--------|-------|-----|
| Firewall rules | **`AR-BLOCK-*` / `AR-INTEL-*`** (contract **1.4.31**) | Kontrollü migrate; wipe hâlâ `HP-*` + legacy siler — [`../agent/firewall-brand-migrate.md`](../agent/firewall-brand-migrate.md) |
| Command signing | `yesnext-chp-v1` **değişmez** | Mevcut imzalar |
| ProgramData (mevcut) | `%ProgramData%\YesNext\CloudHoneypotClient` | Sessiz taşıma yok |
| Scheduled tasks (mevcut) | `CloudHoneypot-*` | Aynı |
| Exe (mevcut) | `honeypot-client.exe` / … | Yeni kurulumda `asteria-client.exe` opsiyonel |
| Contract repo | `asteria-contract` (legacy: `honeypot-contract`) | GitHub rename; redirects preserve history |
| API paths | `/api/...`, `/ws/...` | Path rename yok |

## İzin verilen

- **API base URL** → `https://asteria.run` (legacy host failover)
- Firewall prefix cutover **AR-*** (client ≥ **4.9.33**, küçük filo clear+rewrite)
- UI / installer / tray / e-posta → Asteria; mail subject `· asteria.run`
- Yeni kurulum path `Program Files\Asteria\...`

## Yasak

- Wipe’ta `HP-*`’yi unutup yalnız `AR-*` silmek (orphan)
- `yesnext-chp-v1` signing context değişimi
- ProgramData sessiz taşımak
- Legacy host’u fleet migrate olmadan kaldırmak

## Cloud checklist (shipped 1.4.30)

- [x] nginx `asteria.run` + `www.asteria.run` + legacy host aynı upstream
- [x] Deploy root rename: `/data/asteria.run` (PM2 `asteria-api`)
- [x] `brand.py` + `PUBLIC_BASE_URL` env
- [x] Hardcoded legacy host → `PUBLIC_BASE_URL` (Python)
- [x] UI / i18n / e-posta marka metinleri
- [x] Threat-intel allowlist: her iki host
- [x] Agent path trust: YesNext **ve** Asteria Program Files
- [ ] Cloudflare DNS `asteria.run` → bu origin (operator)
- [ ] Legacy host decommission (fleet migrate sonrası)

## Client

Zorunlu iş listesi: [`../agent/rebrand-asteria.md`](../agent/rebrand-asteria.md) · min client **≥ 4.9.32**.
