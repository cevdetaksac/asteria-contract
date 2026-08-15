# Self-update — single contract

> **SoT** contract **≥ 1.4.61** · Client **≥ 4.9.96** (asset alias) / progress **≥ 4.9.60**
> Pointers: [`../api/04-self-update.md`](../api/04-self-update.md),
> [`../agent/self-update-progress.md`](../agent/self-update-progress.md)

Dashboard **Şimdi güncelle** → `self_update` (TTL 30m). Scheduled auto-update is separate.

```json
{
  "command_type": "self_update",
  "params": {
    "force": true,
    "tag": "v4.9.96",
    "download_url": "https://github.com/cevdetaksac/asteria-client/releases/download/v4.9.96/asteria-client-installer.exe",
    "installer_name": "asteria-client-installer.exe",
    "size": 323000000,
    "triggered_by": "dashboard"
  }
}
```

| ID | Rule |
|----|------|
| **CL-UPD-TRUST-1** | `signed=null` / `checksum_valid=null` / “trust metadata pending” = **observe**. Must not reject download. |
| **CL-UPD-TRUST-2** | Missing Authenticode must not abort installer download. |
| **CL-UPD-ASSET-1** | GitHub Release binary: **only** `asteria-client-installer.exe`. |
| **CL-UPD-ASSET-2** | `installer_name` + `download_url` point at that file. Client rewrites legacy `cloud-client-installer.exe`. |
| **C-UPD-PROG-1** | Download ticks ≥ every 2s (`phase`, `progress_pct`, `bytes_*`) |
| **C-UPD-PROG-2** | Silent `running` >5s without a tick = FAIL |
| **C-UPD-CHICKEN** | Hosts on ≤4.9.72 need **one elevated land**; cloud cannot patch the old helper. |

Phases: `queued` → `downloading` → `verifying` → `installing` → terminal `completed`/`failed`.
Single-flight gate: overlapping Update now returns the in-flight snapshot.

`check_update` is inspect-only (no install).
