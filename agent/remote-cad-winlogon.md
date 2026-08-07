# Remote CAD + Winlogon input honesty (P0)

> **Contract VERSION:** **1.4.54** (opened **1.4.52**)  
> Status: **Meta accepted — client ≥ 4.9.86** (lab Derin-Web 2026-08-07)  
> Related: [`remote-input.md`](./remote-input.md) ·
> [`winlogon-session0-capture.md`](./winlogon-session0-capture.md) ·
> [`../cloud/remote-console-parity.md`](../cloud/remote-console-parity.md) ·
> [`../api/05-remote-desktop.md`](../api/05-remote-desktop.md)

## Lab evidence

### Fail honesty baseline (**4.9.84**)
`success:true` / `SendSAS(0) called` while CAD tip unchanged → false positive.

### Honesty pass (**4.9.85**)
`SAS_NO_EFFECT` + `inputs_applied` ✅; `software_sas_generation:null`, `ui_*:unknown`, no SAS UI.

### Meta close (**4.9.86** — Derin-Web)

Release: https://github.com/cevdetaksac/asteria-client/releases/tag/v4.9.86

| Field | Lab value |
|-------|-----------|
| `success` | `true` |
| `message` | `SendSAS effect observed` |
| `path` | `service` |
| `as_user` | `false` |
| `software_sas_generation` | **`1`** (int; `policy_note` enabled/policy_ok) |
| `ui_before` / `ui_after` | **`sas_ui` / `sas_ui`** (not `unknown`) |
| `effect` | `true` |
| `execution_time_ms` | ~46–63 |
| Stream | `persistent-winlogon-helper:raw`, `streaming:true` |

**Residual (non-blocking):** JPEG/WS viewport sampled solid blue (`avg≈(1,88,157)`, `brightRatio=0`) while agent reports `sas_ui` — lock wallpaper/CAD tip chrome not visible in lab frames. Meta contract rows closed; optional capture polish for readable SAS chrome on stream.

---

## Cloud wire (CL-RD-CAD — shipped with **1.4.52**)

| ID | Rule |
|----|------|
| **CL-RD-CAD-1** | Dashboard CAD uses **only** `POST /api/remote/cad` → `remote_send_sas` |
| **CL-RD-CAD-2** | Winlogon CAD: `prefer=winlogon` / `pre_logon`; omit username + forced SID |
| **CL-RD-CAD-3** | No synthetic `key=ctrl+alt+delete` as SAS substitute |
| **CL-RD-CAD-4** | Surface command result honestly (`SOFTWARE_SAS_DISABLED` / `SAS_NO_EFFECT` / ok) |

---

## Client MUST — CAD (C-RD-CAD-*) — **≥4.9.86**

| ID | Rule |
|----|------|
| **C-RD-CAD-1** | Raise SAS on stream console; prefer visible security/password UI |
| **C-RD-CAD-2** | Session-0: service `SendSAS(FALSE)` with console affinity; helper / impersonate fallbacks |
| **C-RD-CAD-3** | Ensure `SoftwareSASGeneration` ∈ `{1,3}`; result always int `0..3`; closed → `SOFTWARE_SAS_DISABLED` |
| **C-RD-CAD-4** | No success on VOID alone; tip unchanged → `SAS_NO_EFFECT` |
| **C-RD-CAD-5** | Result: `session_id`, `as_user`, `software_sas_generation`, `ui_before`/`ui_after`, honest `detail` |
| **C-RD-CAD-6** | Ignore synthetic `key=ctrl+alt+delete` (`cad_key_ignored`) |

---

## Client MUST — Winlogon input (C-RD-IN-WL-*)

| ID | Rule |
|----|------|
| **C-RD-IN-WL-1** | Winlogon stream input → same capture helper session |
| **C-RD-IN-WL-2** | Protocol-2 `input_ack.success` = helper inject |
| **C-RD-IN-WL-3** | Meta: `inputs_applied` / `last_input_event` |
| **C-RD-IN-WL-4** | After SAS, Tab/Enter/`type_text` reach credential UI without second Start |

---

## Acceptance (lab)

- [x] Honesty: no false `SendSAS ok` only — **4.9.85+**  
- [x] `software_sas_generation` int (lab **1**) — **4.9.86**  
- [x] `ui_before` / `ui_after` ≠ `unknown` (lab `sas_ui`) — **4.9.86**  
- [x] `success` + `effect` when secure-attention state observed — **4.9.86**  
- [x] Meta `inputs_applied` on Winlogon stream — **4.9.85+**  
- [ ] Optional: operator-visible password/SAS chrome (non-flat capture) on JPEG/WS  

**P0 meta path closed at 4.9.86 / contract 1.4.54.**
