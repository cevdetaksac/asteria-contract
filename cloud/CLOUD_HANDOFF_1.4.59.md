# Cloud handoff — contract 1.4.59 (tag `v1.4.59`)

> Target agent **≥4.9.95**. Lab **4.9.94** follow-skip is incomplete.
> Tree: https://github.com/cevdetaksac/asteria-contract/tree/v1.4.59
>
> SoT: [`remote-console-parity.md`](./remote-console-parity.md) (**C-RD-TOPO**),
> [`../agent/process-inspect.md`](../agent/process-inspect.md),
> [`../api/04-self-update.md`](../api/04-self-update.md),
> [`../api/09-threat-intel.md`](../api/09-threat-intel.md)

## P0-1  `remote_stream_start` is not one shape

Normative: **C-RD-TOPO-1…5**

**A) Default Connect / “Logon · varsayılan”**

`params.topology = "follow"`

**Do not send:** `prefer`, `pre_logon`, `desktop`, `session_id`, `username`

Lab 4.9.93 FAIL: `prefer=winlogon` → Winlogon helper + `SESSION0_HELPER_SPAWN_FAILED`
+ jpeg=0B (console already administrator Active). Follow DXGI/NVENC Default.

```json
{
  "command_type": "remote_stream_start",
  "params": { "topology": "follow", "stream_id": "…", "fps": 12 }
}
```

**B) Logon / Lock row** (empty host, lock, SAS)

`params.topology = "winlogon"`
`prefer: "winlogon"`, `pre_logon: true`, `desktop: "Winlogon"`
`session_id` omitted, `username` omitted. Helper is legitimate.

**C) User shortcut:** `session_id` + `username`. Do not auto-pick first Active SID.

**D) After Enter:** same `stream_id`; no second Start; no “pick administrator”;
`WTSGetActiveConsoleSessionId` → `WinSta0\Default` (C-RD-FOLLOW).
Winlogon spawn while Default is live must not be a terminal FAIL.

Min: **≥4.9.95**

## P0-2  `inspect_process`

- Add to command catalog (`VALID_COMMAND_TYPES`). Not destructive.
- Confirm **no**. “Onayla ve Gönder” / required `confirm:true` = FAIL.
- Dashboard **?** / İncele → immediately `{ "pid": N }` (int > 0).
- Image name `rundll32.exe` is not Kritik/lolbin.
- `dll,Entry` is normal. lolbin only: `http` / `javascript:` / UNC `\\`
- DLL outside System32 → `lolbin_off_system_dll` (İncele; not auto-Kritik).
- Health `top_processes` stays lean; heavy evidence only from inspect.

## P0-3  Trust metadata pending must not block download

`signed=null` / `checksum_valid=null` / “trust metadata pending” = **observe**.
`self_update` GET / GitHub asset / dashboard download is not rejected.
Missing Authenticode must not abort installer download.

## P0-4  Release asset

GitHub Release downloadable binary: **only** `asteria-client-installer.exe`.
Do not publish `cloud-client-installer.exe` / extra exes.
`self_update` `installer_name` + `download_url` must point at that file.

## Additive (not required) — Threat Intel

ACK `stats.firewall_current` (int, standing `AR-INTEL-*` count) optional ≥4.9.94.
No ACK on **304**. ACK on **200**. External ThreatFox/CISA fetch is cloud-side.

## Acceptance

Default Connect follow + logged-on console DXGI frames (jpeg>0),
Lock row Winlogon UI, inspect without confirm, single installer asset.
