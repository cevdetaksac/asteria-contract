# Firewall brand migrate — `HP-*` → `AR-*`

> **Contract:** **1.4.31** · Min client: **≥ 4.9.33**  
> SoT wire names: [`../api/06-firewall-blocks.md`](../api/06-firewall-blocks.md) · intel: [`../api/09-threat-intel.md`](../api/09-threat-intel.md)  
> Brand: [`../cloud/PRODUCT_BRANDING.md`](../cloud/PRODUCT_BRANDING.md)

Küçük filo (birkaç sunucu) için **kontrollü** prefix cutover. Big-bang yok;
client hem eski hem yeni önekleri **siler**, **yeni kuralları yalnız `AR-*`** yazar.

---

## Yeni wire kimliği

| Kaynak | Eski (legacy, silinir) | Yeni (yazılır) |
|--------|------------------------|----------------|
| Auto / dashboard / `block_ip` | `HP-BLOCK-*`, `HONEYPOT_BLOCK_*` | **`AR-BLOCK-{ip}`** |
| Threat intel | `HP-INTEL-*` | **`AR-INTEL-{id}`** |

Örnek:

- `AR-BLOCK-203.0.113.10`
- `AR-INTEL-tf-1.2.3.4` (id cloud bundle’daki IoC id)

Inbound (+ intel için outbound) davranışı **değişmez** — yalnız isim öneki.

---

## Client algoritması (≥ 4.9.33)

### 1) Yeni kural yazarken

1. Auto-block / pending-blocks / `block_ip` → **yalnız** `AR-BLOCK-{ip}` oluştur.
2. Threat-intel apply → **yalnız** `AR-INTEL-{id}` oluştur.
3. Aynı IP için eski `HP-BLOCK-{ip}` varsa önce sil, sonra AR yaz (idempotent).

### 2) Silme / wipe

`clear_firewall` veya `unblock_ip` / whitelist enforce:

Aşağıdaki **tüm** önekleri sil (netsh name filter / enum):

| Prefix |
|--------|
| `AR-BLOCK-` |
| `AR-INTEL-` |
| `HP-BLOCK-` |
| `HP-INTEL-` |
| `HONEYPOT_` (legacy) |
| `CloudHoneypot` (varsa display/legacy) |

`wipe_all_honeypot_rules: true` (geriye dönük ad) = yukarıdaki listenin tamamı.  
Yeni param (tercih): `wipe_prefixes: ["AR-BLOCK","AR-INTEL","HP-BLOCK","HP-INTEL","HONEYPOT"]`.

`unblock_ip` için tek IP: `AR-BLOCK-{ip}` **ve** `HP-BLOCK-{ip}` (ve legacy adlar) dene; hangisi varsa sil.

### 3) Boot / self-update migrate (zorunlu bir kez)

Client ≥ 4.9.33 ilk çalıştırmada (veya `migrate_firewall_brand` komutu gelince):

1. Enumerate local `HP-BLOCK-*` / `HP-INTEL-*`.
2. Her `HP-BLOCK-{ip}` → `AR-BLOCK-{ip}` yoksa oluştur → HP sil.
3. Her `HP-INTEL-{id}` → `AR-INTEL-{id}` yoksa oluştur → HP sil.
4. `POST /api/agent/sync-rules` (`mode=snapshot`) ile buluta bildir.
5. Marker: ProgramData ayarında `firewall_brand=ar` / `firewall_brand_migrated_at=…` — tekrar etme.

Cloud ayrıca fleet helper ile `clear_firewall` + pending re-issue gönderebilir
(aşağıdaki cloud akışı); client iki yolu da güvenli karşılamalı.

### 4) `rule_name` / `firewall_rule_name` raporlama

ACK / auto-block / sync gövdelerinde gönderilen isim **`AR-BLOCK-…` / `AR-INTEL-…`** olmalı.
Cloud isim doğrulamaz (IP SoT); UI’da “Asteria engeli” gösterilir.

---

## Cloud akışı (filo migrate)

Operator / dashboard:

1. Client ≥ **4.9.33** deploy (self-update).
2. Cloud: `clear_firewall` wipe (HP+AR+legacy sil).
3. Cloud: mevcut `BlockRule` satırlarını `pending` yap + isteğe `block_ip` push
   → agent `AR-BLOCK-*` yazar.
4. Sonraki threat-intel poll → `AR-INTEL-*`.
5. `sync-rules` → bulut envanter temiz.

Script / endpoint: `scripts/migrate_firewall_ar_prefix.py` ·
`POST /api/premium/migrate-firewall-brand` (dashboard auth).

---

## Yasak

- Yalnız `AR-*` yazıp wipe’ta `HP-*`’yi unutmak (orphan HP kuralları kalır).
- `yesnext-chp-v1` signing context değiştirmek (bu migrate’in parçası değil).
- ProgramData sessiz taşımak.

---

## Acceptance

- [ ] Yeni block → netsh’te `AR-BLOCK-*`, `HP-BLOCK-*` yok
- [ ] Intel apply → `AR-INTEL-*`
- [ ] `clear_firewall` wipe → HP **ve** AR **ve** legacy yok
- [ ] `unblock_ip` eski HP kuralını da kaldırır
- [ ] Boot migrate: HP→AR, sync 200
- [ ] Cloud migrate helper: clear + pending re-apply → agent AR yazar
- [ ] Filtre mailleri: subject sonunda `· asteria.run` (ayrı mail teması)

---

## Sürüm

- Contract **1.4.31**
- Client **≥ 4.9.33**
