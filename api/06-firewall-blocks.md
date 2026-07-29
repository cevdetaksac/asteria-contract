# Firewall blocks & sync

> **Contract VERSION:** root `VERSION` (**1.4.31+** AR-* prefix; see rebrand for signing)  
> **Auth:** Bearer  
> **İlgili:** [`../agent/register-protection.md`](../agent/register-protection.md) · threat-intel `AR-INTEL-*` → [`09-threat-intel.md`](./09-threat-intel.md)  
> **Brand migrate:** [`../agent/firewall-brand-migrate.md`](../agent/firewall-brand-migrate.md) (client ≥ **4.9.33**)

Üç kaynak (karıştırma):

| Kaynak | Kural adı (yeni) | Legacy (wipe/unblock siler) | Nasıl gelir |
|--------|------------------|-----------------------------|-------------|
| Dashboard / cloud eval (`notification_rules`) | **`AR-BLOCK-*`** | `HP-BLOCK-*`, `HONEYPOT_*` | `pending-blocks` / `block_ip` |
| Local threat engine auto-block | **`AR-BLOCK-*`** | `HP-BLOCK-*` | Agent netsh + `POST /api/alerts/auto-block` |
| Threat intel bundle | **`AR-INTEL-*`** | `HP-INTEL-*` | `GET /api/agent/threat-intel` apply (client ≥ **4.9.7**; brand ≥ **4.9.33**) |

`ip_or_cidr`: tek IP, CIDR, veya `country:XX` (cloud destekliyorsa).

**Dashboard Firewall Yönetimi** (full inventory / profiles): see
[`../agent/firewall-management.md`](../agent/firewall-management.md) (`list_firewall`,
cloud page `/dashboard/server/firewall`, contract ≥**1.4.40** / client ≥**4.9.40**).

### Whitelist — asla engelleme (client ≥ 4.9.7)

`threats/config.whitelist_ips` / `whitelist_subnets`:

- `block_ip` (auto / komut / intel) whitelist IP’ye **uygulanmaz**.
- Whitelist güncellenince veya block denemesi whitelist’e çarparsa client
  mevcut `AR-BLOCK-*` / `HP-BLOCK-*` **ve** eşleşen `AR-INTEL-*` / `HP-INTEL-*`
  kurallarını **derhal siler** (`enforce_whitelist_unblocks`).
- Bare `successful_logon` zaten BLOCK üretmez → [`../agent/threat-engine.md`](../agent/threat-engine.md).

---

## Whitelist invariant (contract ≥1.4.11)

`threats/config.whitelist_ips` / `whitelist_subnets` (ve silent-hours WL) içindeki
IP **asla** engelli kalmamalı.

| Kim | Ne yapar |
|-----|----------|
| Client | Auto-block / manual block öncesi WL kontrolü; WL ise AR/HP-BLOCK yok |
| Cloud | `POST /api/alerts/auto-block` → `rejected`/`whitelisted`; manuel block-rule create → reject |
| Cloud lift | BlockRule `remove_pending` + AutoBlock pasif + `unblock_ip` komutu + `GET pending-unblocks` |
| Client | `unblock_ip` **ve/veya** pending-unblocks → netsh sil (AR **ve** HP adları) → `POST block-removed` |

Reconcile tetikleri (cloud): whitelist-add, `GET/POST /api/threats/config`,
`sync-rules`, cleanup sweep.

Control WS (opsiyonel push):

```json
{ "v": 1, "t": "pending_unblocks_updated", "ips": ["84.44.42.18"], "reason": "whitelist_guard" }
```

Agent: mesaj gelince hemen `GET /api/agent/pending-unblocks` (poll bekleme).

---

## Agent HTTP

| Method | Path | Ne |
|--------|------|-----|
| GET | `/api/agent/pending-blocks` | `status=pending` → uygula (`AR-BLOCK-*`) |
| POST | `/api/agent/block-applied` | `{ block_ids[] }` veya `ip` + `rule_name` |
| GET | `/api/agent/pending-unblocks` | Kaldırılacaklar |
| POST | `/api/agent/block-removed` | ACK |
| POST | `/api/agent/sync-rules` | Canlı FW envanteri → cloud |
| POST | `/api/premium/clear-all-blocks` | Dashboard; komut `clear_firewall` push |
| POST | `/api/premium/migrate-firewall-brand` | Wipe legacy HP + re-pend blocks as AR (dashboard) |

### block-removed ACK (client ≥4.8.5)

Body (agent → cloud):

```json
{ "token": "…", "block_ids": [2582, 2649], "ips": ["50.16.16.211", "178.62.3.223"], "ip": "…" }
```

- `block_ids`: pending-unblocks öğelerindeki `id` (tercihen int).
- `ips` / `ip`: aynı satırların IP'leri — **zorunlu pratik alan**.
- Client ≥4.8.5 her iki alanı birden gönderir; `updated=0` ise IP başına retry yapar.
- Cloud `block_ids` **ve** `ips`/`ip` birlikte değerlendirilir.

### sync-rules (özet)

Agent `netsh` / `AR-BLOCK-*` (+ envanterde `AR-INTEL-*`; migrate sırasında
geçici `HP-*` görünürlüğü) taraması → JSON blocks listesi. Cloud SoT = IP.
İsim önekleri karıştırılmaz: yeni yazım yalnız **AR-***.

### clear_firewall (komut)

Params:

```json
{
  "wipe_all_honeypot_rules": true,
  "wipe_prefixes": ["AR-BLOCK", "AR-INTEL", "HP-BLOCK", "HP-INTEL", "HONEYPOT"],
  "delete_each": true,
  "mode": "per_ip_then_wipe",
  "ips": ["…"],
  "reason": "…",
  "immediate": true
}
```

- `wipe_all_honeypot_rules: true` (geriye dönük) = tüm Asteria/legacy öneklerini sil.
- `wipe_prefixes` (≥4.9.33): açık liste; yoksa client wipe_all ile aynı seti kullanır.
- Wipe **AR-BLOCK / AR-INTEL / HP-BLOCK / HP-INTEL / HONEYPOT_*** siler.
- Sonra sync-rules + isteğe clear-data scopes.

---

## Premium notification rules

Dashboard CRUD: `GET/POST/PUT/DELETE /api/premium/rules`  
Seed: `POST /api/premium/rules/seed-defaults` (dashboard login).  
Yeni client: register otomatik seed ([`register-protection.md`](../agent/register-protection.md)).

Agent eşikleri **local** `protection.block_rules` + cloud worker pending-blocks ile çift katman olabilir; idempotent block.

---

## Acceptance

- [ ] pending-block → **AR-BLOCK** → block-applied 200  
- [ ] clear_firewall wipe → sync boş / cloud clear (**AR-INTEL + HP-*** dahil)  
- [ ] INTEL kuralları BLOCK ile isim çakışması yok (`AR-INTEL-*` ≠ `AR-BLOCK-*`)
- [ ] Whitelist IP → block yok; engelli ise anında kaldırılır (client ≥4.9.7)
- [ ] Whitelist IP auto-block denemesi → cloud `rejected`/`whitelisted` + client FW’de kural yok
- [ ] Whitelist’e eklenen önceden bloklu IP → `remove_pending` + `unblock_ip` / pending-unblocks → `block-removed` ACK `updated>0`
- [ ] Çıplak `successful_logon` auto-block → cloud `successful_logon_no_autoblock` reject
- [ ] Brand migrate ≥4.9.33: HP→AR ([`firewall-brand-migrate.md`](../agent/firewall-brand-migrate.md))
