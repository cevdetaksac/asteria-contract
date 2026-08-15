# Anti-brick — account-gated critical actions

> **Behavior SoT:** [`../features/account-safety.md`](../features/account-safety.md) (**≥ 1.4.61**). Appendix below.

> **Contract:** **1.4.38** (was 1.4.34; undo_mail_path wire)  
> **Min client:** **≥ 4.9.46** (1.3 probe + rolled_back alert; floor was 4.9.36)  
> **Cloud:** silent-hours defaults OFF + dashboard auto-link + undo mail + **`undo_mail_path` on account-status** (this doc + [`../cloud/CLOUD_CHECKLIST.md`](../cloud/CLOUD_CHECKLIST.md))  
> **Related:** [`api/02-account.md`](../api/02-account.md) · [`threat-engine.md`](./threat-engine.md) · [`server-management.md`](./server-management.md) · [`disaster-recovery.md`](./disaster-recovery.md)

## Problem (incident class)

Yerel otomatik tepki (`silent_hours` → `logoff_user` + `disable_account` on **Administrator**) hesap bağlantısı / dashboard kurtarma yolu yokken uygulandı → sunucu brick (RDP/admin kilit).

**Kural özeti:** Operatörün dashboard’dan (veya mail undo ile) geri alamayacağı hiçbir kritik yerel adım atılmaz.

---

## Definitions

| Terim | Anlam |
|-------|--------|
| **Account-linked** | `AccountClient` satırı var; `GET /api/agent/account-status` → `account_linked: true` |
| **Orphan / unlinked** | Token geçerli client satırı ama hiç hesaba bağlı değil |
| **Critical local auto-action** | Operatör onayı olmadan client’ın SAM / oturum / lockdown değiştiren tepkisi |
| **Admin-class account** | Built-in **Administrator** (RID-500) veya `Administrators` grubundaki son aktif yerel admin |

### Critical local auto-actions (gated)

Şunlar **hesap bağlı değilse yasak** (yalnızca alert / observe):

- `disable_account` (özellikle admin-class)
- `disable_all_users` (panic)
- Silent-hours / time-rule: `auto_disable_account`, `auto_logoff` (admin-class veya tek admin yolu)
- `enable_lockdown` / network isolate that removes management path without break-glass
- Herhangi bir yol ki **son admin-class hesabı** disabled bırakır

**İzinli (unlinked iken):** `block_ip` (whitelist kurallarına uyarak), alert/`notify_urgent`, intel observe, non-destructive list komutları.

---

## C-BRICK-1 — Client: account gate (fail-closed)

1. Daemon kritik yerel auto-action öncesi **taze** bağ kontrolü yapar:
   - Kaynak: `GET /api/agent/account-status` (Bearer token)
   - Cache TTL ≤ **15 dakika**; expire → yeniden GET
   - GET başarısız / timeout / `account_linked != true` → **aksiyonu atla** (fail-closed)
2. Atlanan aksiyon için urgent/alert:
   - `threat_type`: `critical_action_skipped_unlinked` (veya mevcut `silent_hours_violation` + `auto_response_taken: ["notify_urgent","skipped_unlinked"]`)
   - `recommended_action`: hesabı bağla / dashboard’dan onayla
3. `account_linked: true` iken bile admin-class **auto-disable** yalnızca:
   - effective config açıkça izin veriyorsa **ve**
   - en az bir **break-glass** yolu varsa (başka aktif local admin **veya** cloud undo-mail path canlı — C-BRICK-5)
4. Clone / yeni `client_id` (machine_id split): yeni satır **unlinked** başlar → kritik auto **kapalı** ta ki link oluşana kadar.

### Command result status (wire fix)

`POST /api/commands/result` alanındaki `status` **komut durumu**dur: `completed` / `failed` / `rejected` / …

- Hesap durumu (`active` / `disabled`) **yalnız** `result.data` / `result.enabled` içinde gider.
- Client `status: "active"` göndermez (cloud 1.4.34+ alias ile tolere eder; client ≥4.9.36 düzeltir).

---

## C-BRICK-2 — Defaults: time rules OFF

Yeni `threat_config` / client first-sync:

| Alan | Default |
|------|---------|
| `silent_hours.enabled` | **false** |
| `silent_hours.weekend_all_day_silent` | **false** |
| `silent_hours.auto_block_ip` | **false** |
| `silent_hours.auto_logoff` | **false** |
| `silent_hours.auto_disable_account` | **false** |
| `working_hours.enabled` | **false** |
| `logon_challenge_enabled` | **false** (zaten) |

Operatör dashboard’dan **bilinçli** açmadan bu kurallar asla “surprise ON” olmaz.  
Fleet migrate: cloud mevcut satırlarda da OFF’a çekebilir (ops); yeni default şema `DEFAULT 0`.

GUI Ayarlar örneği zaten `silent_hours.enabled: false` göstermeli ([`gui-control-center.md`](./gui-control-center.md)).

---

## C-BRICK-3 — Cloud dashboard: auto-link on authenticated token use

**Amaç:** Client bağlı / dashboard “görünmüyor” veya hesap–token kopukken bile, giriş yapmış operatör token ile geldiğinde sunucu **hemen o hesaba** bağlansın.

### Ne zaman

Aşağıdakilerin **hepsi** doğruysa cloud **idempotent** `AccountClient` oluşturur:

1. İstek **account session** ile authenticated (dashboard cookie / account Bearer — agent token değil).
2. İstek bir **agent client token** taşıyor (`?token=`, body `token`, veya server-scoped path).
3. Token geçerli bir `Client` satırına çözülüyor.
4. Bu account ↔ client için membership **yok**.

### Davranış

| Durum | Sonuç |
|-------|--------|
| Unlinked | Bu account’a link et; `linked_at=now`; audit `auto_link_dashboard` |
| Zaten bu account’a linked | no-op |
| Başka account’a linked | **Çalma yok** — `409` / UI: “Başka hesaba bağlı” + transfer/unlink akışı |

### UX

- `/dashboard?token=…` veya Servers / Users / Remote açılışı auto-link tetikleyebilir.
- Link sonrası Servers listesinde anında görünür; orphan twin (eski `client_id`) banner ile işaretlenir.

Normative API notları: [`api/02-account.md`](../api/02-account.md) § Auto-link (1.4.34).

---

## C-BRICK-4 — Remote commands still require linkage for destructive IR (cloud)

Dashboard / `POST /api/commands/send` yıkıcı IR (`disable_account`, `disable_all_users`, `enable_lockdown`, …):

- İstek account session’ından geliyorsa: client bu account’a linked olmalı (değilse C-BRICK-3 auto-link dene; yine yoksa **403**).
- Agent-poll komutları zaten token-scoped; orphan client’a operatör komut gönderemez çünkü UI listede yok — auto-link bunu düzeltir.

---

## C-BRICK-5 — Undo mail + recovery key (cloud, required for admin-class disable)

Her **başarılı** admin-class disable sonrası (kaynak: client auto-response **veya** dashboard komutu):

1. Linked account `alert_email` / owner email’e **anında** mail.
2. Mail içeriği (min):
   - Sunucu alias / hostname / IP
   - Kullanıcı (`Administrator`)
   - Tetik (`silent_hours_violation` / dashboard / …)
   - **Enable** CTA → signed one-time URL
3. One-time **undo key** (TTL ≤ **72h**, tek kullanımlık):
   - `POST /api/recovery/undo-disable` `{ "key": "…" }` (veya GET redeem)
   - Cloud imzalı `enable_account` kuyruğa alır (`priority: critical`)
4. Key yok / expire → dashboard Users → Enable (account-linked client gerekir).
5. **`GET /api/agent/account-status`** → `undo_mail_path: true` **yalnız** mailer + key path gerçekten canlıyken (1.4.38).

Client auto-disable, undo-mail path cloud’da kapalıysa (`undo_mail_path` false/missing) C-BRICK-1.3 gereği **yine skip** (son admin yolu için).

---

## C-BRICK-6 — Never leave zero admin path

Local auto veya komut uygulandıktan sonra client (mümkünse) doğrular:

- En az bir aktif local admin **veya**
- Cloud’un kabul ettiği break-glass (`exclude[]` on `disable_all_users`, veya yeni `create_user` admin) **veya**
- `undo_mail_path: true` iken bilerek son admin disable (C-BRICK-5 kurtarma)

Aksi halde aksiyonu **geri al** (best-effort `enable_account`) + urgent alert:

- `threat_type`: **`critical_action_rolled_back`**
- `auto_response_taken`: `["critical_action_rolled_back","enable_account"]`

---

## Acceptance checklist

### Client ≥ 4.9.46

- [x] Unlinked iken silent-hours `auto_disable_account` / admin `auto_logoff` **çalışmaz**; alert `skipped_unlinked`
- [x] Linked iken admin-class’ta break-glass / `undo_mail_path` yoksa skip (`skipped_no_break_glass`)
- [x] `account-status` cache ≤15m; fail → skip destructive
- [x] Command result `status` ∈ {completed,failed,…}; hesap state `result` içinde
- [x] Rollback → `critical_action_rolled_back`
- [x] Silent-hours first-run defaults OFF (+ fleet canary AND)

### Cloud ≥ 1.4.38 (P0 backlog)

- [ ] DB + model defaults: silent_hours* OFF
- [x] Authenticated dashboard + token → auto-link (C-BRICK-3); foreign link → no steal
- [x] Admin-class disable → undo mail + key (C-BRICK-5) **E2E** (hooks + redeem live; lab mailer-dependent)
- [x] `account-status.undo_mail_path` reflects real C-BRICK-5 readiness (never hardcode true)
- [ ] `commands/result` `status=active|disabled` → normalize completed (compat)
- [ ] Orphan online twin / unlinked banner on Servers

---

## Out of scope / non-goals

- Whitelist / `block_ip` hardening (ayrı threat-engine kuralları)
- ZT operator keys (design-only docs)
- Otomatik `create_user` without operator (disaster-recovery hâlâ confirm’li komut)
