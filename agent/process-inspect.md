# Process inspect (on-demand) — C-PROC-INSPECT

> **Contract VERSION:** **1.4.59**  
> **Client:** ≥ **4.9.93**  
> Related: [`server-management.md`](./server-management.md) ·
> [`../api/07-lifecycle-sessions.md`](../api/07-lifecycle-sessions.md) ·
> [`../api/03-control-websocket.md`](../api/03-control-websocket.md)

Health `top_processes[]` stays **lean** (PID, name, path, cpu/mem, short
`cmdline` ≤500, `suspicious` / `suspicion_reasons`, `inspectable:true`).
Heavy evidence (full cmdline, parent, rundll32 DLL/export, remote peers,
company) is **not** on the 60s health cadence.

Dashboard: **?** on a process row → queue `inspect_process` → modal.

---

## Heuristic change (list row, still cheap)

`rundll32` + `foo.dll,Entry` is **normal**. Do **not** flag `lolbin` for `.dll,`.

Flag `lolbin` only if cmdline has `http` / `javascript:` / UNC `\\`.

Extra: rundll32/regsvr32 loading a DLL **outside** System32 / Program Files →
`lolbin_off_system_dll`. System32 rundll32 + System32 DLL + no URL → **not**
suspicious. Cloud must **not** map every `rundll32.exe` name to **Kritik**.

Suggested UI:

| Client signals | Dashboard |
|----------------|-----------|
| `suspicious=false` | no Kritik (or muted) |
| `lolbin` or `lolbin_off_system_dll` | **İncele** (not auto-Kritik until inspect) |
| inspect `verdict=likely_benign` | clear / “muhtemel yanlış alarm” |
| inspect `verdict=needs_review` | Kritik + modal evidence |

---

## Command `inspect_process`

**Not destructive.** Whitelist `VALID_COMMAND_TYPES`. Confirm gate: **no**.
Dashboard MUST NOT show “Onayla ve Gönder” / generic destructive confirm.

Dashboard **?** / İncele → `{ "pid": N }` immediately.  
**FAIL:** generic “Onayla ve Gönder” / `confirm:true` required. Inspect must queue without a confirm modal.

**Params:**

```json
{ "pid": 5860 }
```

`pid` zorunlu (int > 0).

**Result `data` (success):**

```json
{
  "ok": true,
  "pid": 5860,
  "name": "rundll32.exe",
  "path": "C:\\Windows\\System32\\rundll32.exe",
  "username": "WIN-HOST\\Administrator",
  "ppid": 1234,
  "parent_name": "explorer.exe",
  "parent_path": "C:\\Windows\\explorer.exe",
  "cwd": "C:\\Windows\\System32",
  "cmdline": "C:\\Windows\\System32\\rundll32.exe shell32.dll,Control_RunDLL",
  "started_at": "2026-08-04T01:00:00Z",
  "runtime_sec": 997200,
  "image_system32": true,
  "company": "Microsoft Corporation",
  "suspicious": false,
  "suspicion_reasons": [],
  "rundll32": {
    "dll_path": "shell32.dll",
    "dll_export": "Control_RunDLL",
    "url": "",
    "dll_in_system32": true,
    "dll_company": "Microsoft Corporation"
  },
  "remote_peers": [],
  "peer_count": 0,
  "verdict": "likely_benign"
}
```

`verdict`: `likely_benign` | `observe` | `needs_review`.

`rundll32` is `null` when the image is not rundll32.

`remote_peers[]`: up to **8** non-loopback inet (ESTABLISHED / SYN_SENT).
No full module list. No DNS dump.

**Errors:** `pid required` · `process_gone` · `access_denied` · `invalid_pid`.

Rate: dashboard **one inspect per click**; client has no extra heavy poll.

---

## Client MUST (C-PROC-INSPECT-*)

| ID | Rule |
|----|------|
| **C-PROC-INSPECT-1** | Health/list rows do **not** include peers / full cmdline / signature pack |
| **C-PROC-INSPECT-2** | `inspect_process` returns the evidence pack for one PID |
| **C-PROC-INSPECT-3** | rundll32 `.dll,Entry` is not `lolbin`; http/javascript/UNC is |
| **C-PROC-INSPECT-4** | List rows set `inspectable: true` |

---

## Cloud MUST (CL-PROC-INSPECT-*)

| ID | Rule |
|----|------|
| **CL-PROC-INSPECT-1** | Add `inspect_process` to `VALID_COMMAND_TYPES` (not destructive) |
| **CL-PROC-INSPECT-2** | Process table: **?** / İncele → dispatch `inspect_process` `{pid}` → modal |
| **CL-PROC-INSPECT-3** | Modal shows cmdline, parent, rundll32 dll/export, peers, verdict |
| **CL-PROC-INSPECT-4** | Do not mark Kritik from image name `rundll32.exe` alone |
| **CL-PROC-INSPECT-6** | **No confirm UI.** Dispatch immediately. Destructive-command modal is a contract FAIL |

---

## Non-goals

- Streaming this pack on every health tick
- Auto kill from inspect
- Full loaded-DLL enumeration
