# Process inspect — single contract

> **SoT** **≥ 1.4.65** · Client **≥ 4.9.93** · Inventory: [`server-management.md`](./server-management.md)

Health `top_processes[]` stays **lean**. Heavy evidence only via `inspect_process`.

`rundll32` + `foo.dll,Entry` is **not** `lolbin`. lolbin only: `http` / `javascript:` / UNC `\\`.
DLL outside System32/Program Files → `lolbin_off_system_dll` (İncele, not auto-Kritik).
Do not map image name `rundll32.exe` to Kritik.

## Command

Not destructive. **Confirm gate: no.** “Onayla ve Gönder” / required `confirm:true` = FAIL.

```json
{ "command_type": "inspect_process", "params": { "pid": 5860 } }
```

`pid` int > 0. Result: cmdline, parent, rundll32 dll/export, peers, `verdict`.

| ID | Owner |
|----|--------|
| **C-PROC-INSPECT-1** | Client: lean health rows; `inspectable:true` |
| **C-PROC-INSPECT-2** | Client: evidence pack for one PID |
| **C-PROC-INSPECT-3** | Client: `.dll,Entry` not lolbin |
| **CL-PROC-INSPECT-1** | Cloud: whitelist `inspect_process` |
| **CL-PROC-INSPECT-2** | **?** / İncele → dispatch immediately |
| **CL-PROC-INSPECT-6** | Cloud: **No confirm UI** |

## Client verify

Tick [`CLIENT_CHECKLIST.md`](./CLIENT_CHECKLIST.md) § Process inspect. Floor **≥ 4.9.93**.
