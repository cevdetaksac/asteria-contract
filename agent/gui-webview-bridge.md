# GUI WebView bridge — draft (asteria-gui.exe ↔ motor)

> **Status:** DRAFT — not yet normative. Promote with contract VERSION bump.  
> **Parent plan:** client `docs/ASTERIA_DUAL_TRACK_ROADMAP.md` Track G  
> **UX SoT:** [`gui-control-center.md`](gui-control-center.md)  
> **Arch:** [`../api/08-architecture.md`](../api/08-architecture.md)

## Decision

| | |
|--|--|
| Motor | `asteria-client.exe` — SYSTEM; no GUI toolkit |
| GUI | `asteria-gui.exe` — interactive session; WebView2 + React |
| Transport | TCP `127.0.0.1:58632` existing daemon IPC (allowlisted subset) |
| Secrets | Token / PIN hash / machine_id **never** exposed to JS |

## Process rules

1. GUI crash must not stop honeypots / firewall / control-WS.
2. Tray may launch/focus `asteria-gui.exe`; GUI does not own tray lifecycle.
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

## Open questions

1. Named pipe + ACL instead of (or in addition to) TCP :58632 for GUI-only?
2. Mutual auth between gui.exe and motor (per-install secret in ProgramData)?
3. Single installer artifact vs optional GUI-less server SKU?

## Min client

Target: dual-exe lab ≥ **4.10.0-beta**; GA with dual-track roadmap **4.11.0**.
