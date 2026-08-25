# Attacks & honeypot services

> **Contract VERSION:** root `VERSION`  
> **API base:** `https://asteria.run`  
> **Auth:** Bearer (register hariç)  
> **Kaynak:** canlı `routes_agent.py` + `schemas.py`

Client kendi içinde bait servisler çalıştırır (RDP/SSH/FTP/MYSQL/MSSQL); credential yakalar → cloud’a POST. Gerçek auth **yapılmaz**.

---

## POST /api/attack

Alias’lar (aynı handler ailesi): `/api/honeypot-attack`, legacy `/attacks/` (varsa).

```json
{
  "token": "<optional if Bearer>",
  "attacker_ip": "203.0.113.10",
  "target_ip": "198.51.100.5",
  "username": "administrator",
  "password": "Passw0rd!",
  "service": "RDP",
  "port": 3389,
  "source": "honeypot",
  "logon_type": 3,
  "auth_package": "Negotiate",
  "logon_process": "User32",
  "status": "0xC000006D",
  "substatus": "0x0",
  "workstation": "-"
}
```

| Alan | Not |
|------|-----|
| `service` | `RDP` / `SSH` / `FTP` / `MSSQL` / `MYSQL` / `NETWORK` (`Network`) — **client SoT**; cloud remap yok |
| `port` | Gerçek hedef; biliniyorsa **`0` yasak** (RDP→3389, SMB/Network→445) |
| `source` | `honeypot` (bait) vs `eventlog` (gerçek 4625) — kanallar karışmaz |
| EventLog zenginleştirme | `logon_type`, `auth_package`, `logon_process`, `status`/`substatus`, `workstation` (4625) |
| `attacker_ip` | Yoksa `ip` / peer IP |
| Şifre | Bait: yakalanan; EventLog: `<failed_logon>`. Cloud log/audit’te maskelenebilir |

**200:** `{ "status": "ok", … }` — cloud `attacks` satırı + geoenrich + notification_rules eşik sayacı.

### İki kanal (MUST)

| Kanal | Ne zaman | Dashboard |
|-------|----------|-----------|
| **Bait honeypot** | Tuzak listen `started` + protokol credential | `service` porttan (RDP/SSH/…); tuzaklar **stopped** iken RDP/SSH bait satırı **beklenmez** |
| **EventLog real-port** | Security `4625` / TS / MSSQL fail + `protection.block_rules` | Sınıflandırma: [`features/threat-engine.md`](../features/threat-engine.md) — NLA → **RDP:3389**, SMB → **NETWORK:445** |

Operatör notu: **`NETWORK` = EventLog ağ oturumu (LogonType 3, SMB/null)** — bait tuzak durumu ayrı. “Hepsi NETWORK” yanılsaması çoğunlukla RDP NLA’nın yanlış etiketi + bait’lerin kapalı olmasından.

**Cloud:** `normalize_service` client string’ini olduğu gibi yazar; **RDP↔NETWORK remap client SoT**.

---

## Honeypot servis modeli (eski “tunnel”)

Dashboard hâlâ “tunnel” API isimleri kullanır; anlam = **bait listen desired/state**.

| Endpoint | Kim | Ne |
|----------|-----|-----|
| `GET /api/premium/tunnel-status` | Agent poll | Desired + reported state |
| `POST /api/agent/tunnel-status` | Agent | Gerçek listen durumu raporu |
| `POST /api/premium/tunnel-set` | Dashboard | Desired start/stop / port |
| Komut `tunnel_start` / `tunnel_stop` | Control WS / poll | Agent bait servisi aç/kapa |

Varsayılan listen portları (şablon): RDP 3389, MSSQL 1433, MYSQL 3306, SSH 22, FTP 21.

**RDP port taşıma:** Gerçek RDP güvenli porta alınır; bait 3389’da kalabilir. Onay akışı client’ta korunur — cloud sadece desired/state görür.

### `pending_tunnel_commands` yaşam döngüsü + expiry (cloud kuralı)

`GET /api/premium/tunnel-status` yanıtındaki `pending_tunnel_commands[]` **bilgilendirme amaçlı kuyruk**tur; client **tüketmez**. Client reconcile’ı yalnızca `services[].desired` alanına uyar (bkz. Client kuralları #3). Bu nedenle kuyruk cloud tarafında **kendiliğinden temizlenmezse süresiz birikir** (canlı örnek: Eylül 2025 tarihli `start` komutları Temmuz 2026’da hâlâ dönüyordu — 10 ay).

Cloud **zorunlu** davranış (✅ **implemented** — `routes_agent._prune_tunnel_commands`):

| Kural | Detay | Durum |
|-------|-------|-------|
| **Desired otoritesi** | `tunnel-set` / `tunnel_start`/`tunnel_stop` işlendiğinde ilgili `services[].desired` **hemen** güncellenir. Komut kuyruğu tek başına state değildir. | ✅ |
| **TTL / expiry** | `pending_tunnel_commands[i].created_at` yaşı **> `TUNNEL_CMD_TTL`** (**24 saat**, `_TUNNEL_CMD_TTL_MIN`) olan komut yanıttan düşülür; hem yazma (`tunnel-set`) hem okuma (`tunnel-status`) yolunda. | ✅ |
| **Ack / consume** | Agent `POST /api/agent/tunnel-status` ile gerçek durumu bildirince `desired == reported` olan bekleyen komutlar kuyruktan silinir. | ✅ (mevcut) |
| **Dedupe** | Aynı `service` için yalnızca **en yenisi** tutulur (start/stop fark etmez — en son desired kazanır); eskiler düşer. | ✅ |
| **İdempotent** | `tunnel-set` aynı servisin önceki girdilerini silip yeniyi yazar. | ✅ |

> Client tarafı değişmez: `pending_tunnel_commands` **her zaman** yok sayılır, yalnızca `desired` uygulanır (client ≥4.4.44). Bu bölüm tamamen **cloud/API** temizlik sözleşmesidir.

---

## POST /api/agent/open-ports

```json
{
  "ports": [
    { "port": 3389, "protocol": "tcp", "process": "svchost.exe", "state": "LISTEN" }
  ]
}
```

Cadence ~**5 dk**. Cloud `settings_json.open_ports` yazar; dashboard honeypot işaretleri bilinen port setine göre.

---

## Client kuralları

1. Bait protokol handshake + credential capture; başarılı gerçek login **yok**.
2. Her bait yakalamada `/api/attack` (veya batch politikası varsa); `source=honeypot`.
3. Desired tunnel state’e uy; rapor `tunnel-status` ile.
4. `protection.block_rules` / local fail sayacı **gerçek** auth fail event’leri içindir — bait capture ayrı kanal (`agent/register-protection.md`, `features/threat-engine.md`).
5. EventLog `/api/attack`: sınıflandırma + port + zengin alanlar (`threat-engine.md`); empty/anonymous Network spam’i Attacks’e basma.
6. Client **≥ 4.9.108** — lab prod (2026-08-25): ~12k/gün `NETWORK port=0` vs ~16 `RDP:3389` = yanlış etiket + `port:0` bug; bait hepsi stopped.

---

## Acceptance

- [ ] Bait RDP/SSH fail credential → `/api/attack` 200, dashboard Attacks’te görünür (`source` bait)
- [ ] NLA RDP fail (bait off) → Attacks **`RDP` / `3389` / `<failed_logon>`** — not `NETWORK`/`0`
- [ ] Saf SMB/445 fail → **`NETWORK` / `445`**
- [ ] Tunnel-set start → agent listen + tunnel-status `running`
- [ ] open-ports periyodik 200
- [x] Cloud: `tunnel-set` → `services[].desired` anında güncellenir
- [x] Cloud: `created_at` > 24s bekleyen `pending_tunnel_commands` `expired` + yanıttan düşer
- [x] Cloud: `desired == reported` olunca ilgili bekleyen komut `completed`/silinir
