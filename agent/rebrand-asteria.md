# Client rebrand — Asteria (`asteria.run`)

> **Contract:** **1.4.30** · Min client: **≥ 4.9.32**  
> Brand SoT: [`../cloud/PRODUCT_BRANDING.md`](../cloud/PRODUCT_BRANDING.md)  
> Auth: [`../api/01-auth.md`](../api/01-auth.md) · Self-update: [`../api/04-self-update.md`](../api/04-self-update.md)

Cloud **1.4.30** ile primary public origin **`https://asteria.run`**.
Eski host `https://honeypot.yesnext.com.tr` nginx alias olarak kalır —
ama yeni build'ler **primary** olarak `asteria.run` kullanmalı.

---

## P0 — API / WS base URL (zorunlu)

| | |
|--|--|
| **Primary** | `https://asteria.run` |
| **WebSocket** | `wss://asteria.run/ws/agent/control` (+ remote WS aynı host) |
| **Legacy fallback** | `https://honeypot.yesnext.com.tr` (yalnız primary DNS/TLS fail ise) |

### Client algoritması

1. Config / varsayılan `ApiBaseUrl` = `https://asteria.run`.
2. Boot / self-update / register / heartbeat / control-WS hepsi **aynı host**.
3. Primary'ye TCP/TLS/HTTP 5xx-unreachable olursa → legacy host'a **bir kez** failover;
   başarılı olursa oturum boyunca legacy kullan, bir sonraki boot'ta yine primary dene.
4. Kullanıcıya GUI'de görünen "Cloud URL" = primary (legacy'yi gizle).
5. `rotate-token` / `register` response içindeki `dashboard` URL'ini olduğu gibi kullan
   (cloud zaten `asteria.run` döner) — client kendi host'unu hardcoded birleştirme.

### Kabul

- [ ] Temiz kurulum yalnızca `asteria.run` ile register + heartbeat + control WS.
- [ ] Legacy host'a pin'lenmiş eski ajan hâlâ çalışır (cloud alias).
- [ ] Primary down simülasyonunda failover → legacy; recovery sonrası primary'ye dönüş.

---

## P0 — Threat-intel / self-update allowlist

Cloud bundle allowlist'inde **her iki host** var. Client:

- Local allowlist / pinned hosts: `asteria.run`, `www.asteria.run`, `honeypot.yesnext.com.tr`
- GitHub release host'ları değişmedi
- Self-update download URL host'u ne gelirse gelsin; cloud'un verdiği URL'ye güven
  (mevcut imza doğrulama aynen)

---

## P1 — Görünen ad (UI / installer / tray)

| Eski | Yeni |
|------|------|
| YesNext Cloud Honeypot | **Asteria** |
| YesNext | **Asteria** |
| Cloud Honeypot (kısa) | **Asteria** |

- Tray tooltip, About, uninstaller display name, Windows Firewall kural **görünen** adı
  (wire prefix **`AR-BLOCK-*`** / **`AR-INTEL-*`** — contract **1.4.31** / client ≥ **4.9.33**;
  migrate: [`firewall-brand-migrate.md`](./firewall-brand-migrate.md)).
- Start Menu kısayolu: `Asteria`.
- GUI Settings "Account" metinleri: "Asteria account".

---

## P1 — Kurulum path / exe (yeni kurulum)

**Mevcut kurulumlara dokunma** (ProgramData + schtasks + exe path → update kırılır).

| | Mevcut (legacy, koru) | Yeni kurulum (önerilen) |
|--|--|--|
| Install dir | `Program Files\YesNext\Cloud Honeypot\` | `Program Files\Asteria\` |
| ProgramData | `%ProgramData%\YesNext\CloudHoneypotClient\` | **aynı bırak** veya kontrollü migrate planı ayrı MD |
| Exe | `honeypot-client.exe` | `asteria-client.exe` (opsiyonel; her iki ad cloud trust'ta) |
| Tasks | `CloudHoneypot-*` | **aynı bırak** (wire kimliği) |

Cloud path-trust **her iki** vendor dizinini kabul eder (`helpers._path_looks_trusted`).

Self-proof / agent identity HMAC: mevcut `yesnext-chp-v1` context **değişmez**.

---

## P2 — Repo / paket adları (opsiyonel, kırılmaz)

- GitHub release repo adı şimdilik aynı kalabilir
  (`cevdetaksac/yesnext-cloud-honeypot-client`); display title → Asteria.
- Contract repo `honeypot-contract` tarihsel SoT; rename zorunlu değil.
- NuGet / MSI ProductCode değişimi → major upgrade kurallarına uy.

---

## Yasak (client)

1. Wipe/unblock’ta `HP-*` önekini unutmak (orphan) — dual-delete zorunlu (≥ **4.9.33**).
2. `yesnext-chp-v1` signing context değiştirmek.
3. Bare `/register` ile "domain değişti" bahanesiyle yeni token açmak —
   token aynı kalır; sadece API host değişir. Token rekey gerekirse
   [`POST /api/agent/rotate-token`](../api/01-auth.md) (contract **1.4.29**).
4. Eski host'u config'ten tamamen silmek (failover için tut).
5. ProgramData'yı sessizce `Asteria\` altına taşımak (watchdog kaybı).

Firewall prefix cutover (`HP-*` → `AR-*`): [`firewall-brand-migrate.md`](./firewall-brand-migrate.md) (≥ **4.9.33** / contract **1.4.31**).

---

## Test matrisi (client CI / lab)

| # | Senaryo | Beklenen |
|---|---------|----------|
| 1 | Fresh install → `asteria.run` | register ok, dashboard link `asteria.run` |
| 2 | Eski ajan `honeypot.yesnext.com.tr` | heartbeat + WS hâlâ ok |
| 3 | Primary DNS fail → failover | legacy ile sürer; boot'ta primary retry |
| 4 | Self-update over new host | imza ok, servis dirilir |
| 5 | rotate-token after rebrand | same `client_id`, history preserved |
| 6 | Path trust: YesNext **ve** Asteria Program Files | self-proof kabul |
| 7 | Threat-intel GET | 200/304; allowlist hosts apply |

---

## Sürüm

- Contract **1.4.30**
- Client ship target **≥ 4.9.32** (P0 base URL + UI rename minimum)
- P1 path/exe rename ayrı patch olabilir (≥ 4.9.32 veya 4.9.33) — contract bu MD'de takip edilir
