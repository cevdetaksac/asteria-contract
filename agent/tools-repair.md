# Windows Tools Repair — uzaktan OS onarım araç seti

> **Contract VERSION:** root `VERSION` (**1.4.49**)  
> **Min client:** target **≥ 4.9.79** (additive; production floor unchanged)  
> İlgili: [`system-recovery.md`](system-recovery.md) · [`disaster-recovery.md`](disaster-recovery.md) ·
> Control WS: [`../api/03-control-websocket.md`](../api/03-control-websocket.md) ·
> Surface: [`../cloud/CLOUD_SURFACE.md`](../cloud/CLOUD_SURFACE.md)  
> Yerel ikiz: GUI `tools(action, target, confirm)` — [`gui-webview-bridge.md`](gui-webview-bridge.md)

## Amaç

Hasarlı / kilitli Windows hostlarda **allowlist** onarım kartlarını dashboard’dan
tetiklemek. Yürütme **SYSTEM motor** (`asteria-client.exe`) üzerinde olur —
Control Center WebView yalnızca aynı allowlist’i yerel olarak yansıtır.

**Kapsam dışı:**

- Keyfi `cmd` / PowerShell
- Full registry dump / image restore
- Defender zorla enable (v1)
- Etkileşimli MMC “open” (Session-0’da anlamsız) — yalnız yerel GUI

`system_recovery_*` ile örtüşen politika yüzeyi (`policy_restore`, `fix_taskmgr`, …)
için baseline varsa **tercih** `system_recovery_restore`; tools_repair hızlı tek-tık
alias olarak kalır.

## Mimari

```
Dashboard  →  POST /api/commands/send  →  Control WS / pending
                                              ↓
                                    RemoteCommandExecutor (motor, SYSTEM)
                                              ↓
                                    client_windows_tools.run_repair(...)
```

| Katman | Rol |
|--------|-----|
| Cloud | `VALID_COMMAND_TYPES` + soft/destructive confirm gate |
| Motor | Allowlist execute; sonuç `commands/result` |
| GUI | Aynı `action` id’leri; host bridge → aynı Python modülü (veya ileride IPC) |

Motor zaten elevated olduğu için uzak onarımda ayrı UAC gerekmez.
GUI oturumu admin değilse yerel kart `admin_required` dönebilir — uzak yol tercih edilir.

## Komutlar (Control WS)

| `type` | Params | Confirm | Min |
|--------|--------|---------|-----|
| `tools_repair_catalog` | — | hayır | ≥4.9.79 |
| `tools_repair_diagnose` | — | hayır | ≥4.9.79 |
| `tools_repair` | `action`, `confirm?`, `dry_run?` | **mutate†** / soft hayır | ≥4.9.79 |

† Cloud: `tools_repair` `DESTRUCTIVE_COMMAND_TYPES` içinde; soft `action` veya
`dry_run:true` → `confirm:true` **zorunlu değil**. Mutating destructive action
→ `confirm:true` şart.

### Allowlist `action` (v1)

| `action` | Destructive | Grup | Not |
|----------|-------------|------|-----|
| `share_network_fix` | hayır | daily | Konuk auth + discovery + paylaşım FW |
| `printer_fix` | hayır | daily | 0x0000011b / Spooler / kuyruk |
| `audio_fix` | hayır | daily | Audio stack restart |
| `dns_flush` | hayır | daily | |
| `time_sync` | hayır | daily | |
| `auto_fix_findings` | hayır | critical | Son diagnose soft fix’leri |
| `fix_taskmgr` / `fix_regedit` / `fix_cmd` / `fix_shell` | hayır | critical | |
| `restart_explorer` / `restart_taskmgr` | hayır | critical | Explorer Session-0’da sınırlı etki |
| `policy_restore` | hayır | critical | Tercihen `system_recovery_restore` |
| `restart_critical_services` | hayır | services | |
| `webview2` | hayır | runtime | GUI Runtime; motor-only host’ta opsiyonel |
| `icon_cache` / `clear_temp` | hayır | shell | |
| `sfc_scan` / `dism_health` | hayır | deep | Async konsol başlatır |
| `full_safe` | hayır | deep | Sıralı soft paket |
| `winsock_reset` | **evet** | danger | Reboot önerilir |
| `firewall_reset` | **evet** | danger | Asteria FW kuralları da gidebilir |
| `wu_reset` | **evet** | danger | SoftwareDistribution yenile |

Bilinmeyen `action` → reject (`unknown_repair`). Client allowlist dışı id çalıştırmaz.

`status` / `diagnose` yalnız okuma; `tools_repair` ile `action=diagnose` **veya**
ayrı `tools_repair_diagnose` (ikisi de geçerli; diagnose preferred dedicated).

### Envelope

```json
{
  "command_type": "tools_repair",
  "params": {
    "action": "share_network_fix",
    "confirm": false
  }
}
```

Destructive örnek:

```json
{
  "command_type": "tools_repair",
  "params": {
    "action": "winsock_reset",
    "confirm": true
  }
}
```

### Dry-run

```json
{
  "command_type": "tools_repair",
  "params": {
    "action": "firewall_reset",
    "dry_run": true
  }
}
```

```json
{
  "success": true,
  "data": {
    "dry_run": true,
    "action": "firewall_reset",
    "destructive": true,
    "plan": [
      {"step": "netsh", "args": ["advfirewall", "reset"]}
    ]
  }
}
```

`dry_run` v1: client plan özeti döner; **mutate yok**. Desteklenmeyen action’da
`{"dry_run":true,"action":"...","plan":[{"step":"run_repair","note":"no_side_effects_preview"}]}`.

### Catalog / diagnose result (özet)

```json
{
  "success": true,
  "data": {
    "admin": true,
    "issues": 2,
    "critical": 1,
    "findings": [
      {
        "id": "taskmgr_policy",
        "ok": false,
        "severity": "high",
        "detail": "Task Manager disabled by policy",
        "fix": "fix_taskmgr"
      }
    ],
    "repairs": [
      {"id": "share_network_fix", "destructive": false, "group": "daily"}
    ]
  }
}
```

## Güvenlik

- Cloud unknown `command_type` / client unknown `action` → reject.
- HMAC imza varsa doğrula; geçersiz → reject.
- Drift / alert **otomatik** `tools_repair` kuyruğa **almaz** (operatör confirm).
- Result log’a secret / PIN yazılmaz.
- `firewall_reset` Asteria `AR-BLOCK` kurallarını silebilir — dashboard uyarı metni zorunlu.
- `AllowInsecureGuestAuth` / `RpcAuthnLevelPrivacyEnabled=0` bilinçli zayıflatma;
  kart blurb’ında belirtilir (Radmin/VPN/paylaşım senaryosu).

## GUI ikiz

| Yerel | Uzak |
|-------|------|
| `tools("catalog")` | `tools_repair_catalog` |
| `tools("diagnose")` / catalog.diagnose | `tools_repair_diagnose` |
| `tools("repair", action, confirm)` | `tools_repair` `{action, confirm}` |
| `tools("open", id)` | **yok** (yalnız yerel) |

Aynı `action` string’leri. GUI host tercihen motor IPC’ye taşınabilir; kontrat
id’leri değişmez.

## Acceptance

1. `tools_repair_diagnose` → `findings[]` + `issues`  
2. `tools_repair` `dns_flush` without confirm → success  
3. `winsock_reset` without cloud `confirm:true` → 400 / reject  
4. `winsock_reset` + confirm → client detail `winsock_reset_reboot_recommended`  
5. Unknown action → `unknown_repair`  
6. `share_network_fix` elevated motor → guest auth + discovery services attempted  
7. GUI `tools("repair","printer_fix")` aynı allowlist id ile çalışır  
8. Otomatik drift → tools_repair kuyruk **yok**
