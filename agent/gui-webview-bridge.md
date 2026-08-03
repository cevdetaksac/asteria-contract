# GUI WebView bridge — draft (asteria-gui.exe ↔ motor)

> **Status:** DRAFT — not yet normative. Promote with contract VERSION bump.  
> **Parent plan:** client `docs/ASTERIA_DUAL_TRACK_ROADMAP.md` Track G  
> **UX SoT:** [`gui-control-center.md`](gui-control-center.md)  
> **Arch:** [`../api/08-architecture.md`](../api/08-architecture.md)
> **Lab implementation:** client 4.9.34 workspace spike — not released/normative.

## Decision

| | |
|--|--|
| Motor | `asteria-client.exe` — SYSTEM; no GUI toolkit |
| GUI | `asteria-gui.exe` — interactive session; WebView2 + React |
| Transport | TCP `127.0.0.1:58632` existing daemon IPC (allowlisted subset) |
| Secrets | Token / PIN hash / machine_id **never** exposed to JS |

## Process rules

1. GUI crash must not stop honeypots / firewall / control-WS.
2. `asteria-gui.exe` owns the interactive tray; the motor owns only the
   scheduled-task/session launch policy.
3. New IPC opcodes require this MD + tests before client ship.
4. Cloud calls from UI go through GUI host → motor or host-held token proxy —
   never embed agent token in WebView.

## Message sketch (UI ↔ GUI host)

```ts
type HostRequest =
  | { op: "ipc"; cmd: string; args?: unknown; id: string }
  | { op: "cloud"; method: "GET" | "POST"; path: string; body?: unknown; id: string }
  | { op: "shell"; action: "open_dashboard" | "copy_token" | "quit" | "minimize" }
  | { op: "i18n"; lang: "tr" | "en" }
  | { op: "pin"; action: "check" | "set" | "clear"; value?: string };

type HostEvent =
  | { type: "ipc_result"; id: string; ok: boolean; data?: unknown; error?: string }
  | { type: "status_push"; snapshot: unknown }
  | { type: "toast"; level: "info" | "warn" | "error"; message: string }
  | { type: "update_available"; version: string };
```

GUI host translates `op: "ipc"` to the existing control protocol; it does not
invent motor behavior.

## Implemented Control Center allowlist (lab)

Host methods (Python `MotorBridge` / `window.pywebview.api`):

| Method | Notes |
|--------|--------|
| `session` / `unlock` | GuiLock PIN gate |
| `ping` / `status` | Motor health + STATUS snapshot |
| `catalog` | Honeypot port/service table (no secrets) |
| `ipc(cmd, args)` | Allowlisted motor ops only (includes `SELF_UPDATE`) |
| `cloud(method, path, body)` | Host-held token; `threats/config` GET/POST; `alerts/list` GET |
| `pin(action, value, current)` | set / clear / check |
| `shell(action)` | open_dashboard, open_servers, copy_token, open_logs, minimize, quit; **check_updates** → silent motor `SELF_UPDATE` (no browser) |
| `account(action, email, password)` | status / link / unlink (host-held token) |
| `harden(action, target)` | status checks + fix winrm|nla|antivirus |
| `tools(action, target, confirm)` | Local twin of remote `tools_repair*` — catalog/diagnose/repair/open; allowlisted (`agent/tools-repair.md`) |
| `rdp(action, mode)` | status / move secure|rollback |
| `ir(action, username)` | logoff / disable local account |
| `update_banner(action)` | status / dismiss (`update_ui_status.json`) |
| `i18n(lang)` | get/set `tr`|`en` + string table |

**IPC allowlist:** `STATUS`, `THREAT_TOP`, `CLEAR_FIREWALL`, `BLOCK_IP`, `UNBLOCK_IP`,
`RS_STATUS`, `RS_UNLOCK`, `NG_MAINT_*` / `NG_SNAPSHOT` / `NG_ACCEPT_SURFACE`,
`HONEYPOT_LIST` / `HONEYPOT_START` / `HONEYPOT_STOP`, `SELF_UPDATE`.

React Control Center pages: status / threat / iplist / services / layers / settings.
Layers POST uses contract keys: `ransomware_protection_enabled`, `canary_files_enabled`,
`protection.network_guard.enabled`.

Previously the lab host exposed only:

- `session()` — PIN enabled/locked booleans
- `unlock(pin)` — native PBKDF2 verifier with existing lockout policy
- `ping()` — motor availability only
- `status()` — motor `STATUS`, rejected while GUI session is locked

The PIN hash/salt remain native-side in ProgramData. JavaScript sees only the
entered PIN for the duration of the unlock call; it never receives stored
credentials, agent token, or `machine_id`.

## Open questions

1. Named pipe + ACL instead of (or in addition to) TCP :58632 for GUI-only?
2. Mutual auth between gui.exe and motor (per-install secret in ProgramData)?
3. Single installer artifact vs optional GUI-less server SKU?

## Min client

Target: dual-exe lab ≥ **4.10.0-beta**; GA with dual-track roadmap **4.11.0**.
