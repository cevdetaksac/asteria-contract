# Remote CAD + Winlogon input honesty (P0)

> **Contract VERSION:** **1.4.53** (opened **1.4.52**)  
> Status: **Honesty accepted on ≥4.9.85 — SAS UI effect still open (→ ≥4.9.86)**  
> Related: [`remote-input.md`](./remote-input.md) ·
> [`winlogon-session0-capture.md`](./winlogon-session0-capture.md) ·
> [`../cloud/remote-console-parity.md`](../cloud/remote-console-parity.md) ·
> [`../api/05-remote-desktop.md`](../api/05-remote-desktop.md)

## Lab evidence

### Fail honesty baseline (Derin-Web / **4.9.84**)

Capture (C-RD-S0) works: Logon Start → lock UI pixels live over JPEG-WS.

| Action | Cloud / agent | Observed on stream |
|--------|---------------|--------------------|
| Toolbar **CAD** → `remote_send_sas` | `success:true`, `SendSAS ok` / `SendSAS(0) called` | **No change** — CAD tip unchanged |
| Keys / type / click | Toast “gönderildi” | No `inputs_applied` to verify |

**Verdict (4.9.84):** False-positive SAS success.

### Honesty pass + effect fail (Derin-Web / **4.9.85** — 2026-08-07)

| Check | Result |
|-------|--------|
| Agent version | **4.9.85** (auto-updated) |
| CAD result | `success:false`, `error:SAS_NO_EFFECT`, ~2014ms |
| Detail | `path=helper`, `SendSAS(FALSE) invoked; no_sas_effect`, `as_user:false`, `session_id:1` |
| `software_sas_generation` | **`null`** (must report policy int or `SOFTWARE_SAS_DISABLED`) |
| `ui_before` / `ui_after` | **`unknown`** (effect detector weak) |
| Stream meta | `inputs_applied`≥7, `last_input_event` present ✅ (C-RD-IN-WL-3) |
| Visible SAS / password UI ≤2s | **FAIL** (viewport stayed solid / unchanged) |

Release: https://github.com/cevdetaksac/asteria-client/releases/tag/v4.9.85

---

## Cloud wire (CL-RD-CAD — shipped with **1.4.52**)

| ID | Rule |
|----|------|
| **CL-RD-CAD-1** | Dashboard CAD uses **only** `POST /api/remote/cad` → `remote_send_sas` (never typed `ctrl+alt+delete` as the SAS path). |
| **CL-RD-CAD-2** | On Logon/Winlogon stream, CAD params may include `prefer=winlogon` / `pre_logon=true` and **omit** username; **omit forced SID** (same as stream — C-RD-CON-7). |
| **CL-RD-CAD-3** | Do **not** dual-publish a synthetic `key=ctrl+alt+delete` as a substitute for SAS (noise; cannot produce SAS). |
| **CL-RD-CAD-4** | Surface `remote_send_sas` result honestly in UI (failed / `SOFTWARE_SAS_DISABLED` / `SAS_NO_EFFECT`). |

---

## Client MUST — CAD (C-RD-CAD-*)

| ID | Rule |
|----|------|
| **C-RD-CAD-1** | `remote_send_sas` MUST raise SAS on the **same console session** the active remote stream is capturing (Winlogon/lock UI). Visible effect ≤ **2s**: security options **or** credential/password UI (no longer the “Press Ctrl+Alt+Delete” lock tip alone). |
| **C-RD-CAD-2** | Session-0 service: before `SendSAS(FALSE)`, establish **console session affinity** — impersonate a token from `WTSGetActiveConsoleSessionId` (or stream session) / spawn elevated helper **in that session**, then call SendSAS. Do not call SendSAS only in Session 0 with no impersonation and claim success. |
| **C-RD-CAD-3** | Ensure Software SAS policy allows services: `HKLM\\Software\\Microsoft\\Windows\\CurrentVersion\\Policies\\System\\SoftwareSASGeneration` ∈ `{1,3}` (Services or Both). If disabled (`0`) or missing when required → **fail** with `SOFTWARE_SAS_DISABLED` (do **not** `success:true`). |
| **C-RD-CAD-4** | Never report `success:true` solely because `SendSAS()` returned (`VOID`). Prefer: policy check + session-affinity + optional post-condition (desktop/secure desktop change). If API called but no secure-desktop transition within 2s → `SAS_NO_EFFECT` / `success:false`. |
| **C-RD-CAD-5** | Result payload SHOULD include: `session_id` (console used), `as_user` (bool passed to SendSAS), `software_sas_generation` (policy value), `detail` (honest). Drop useless-only `"SendSAS(0) called"` as sole proof. |
| **C-RD-CAD-6** | Ignore synthetic `key=ctrl+alt+delete` for producing SAS; CAD command path only. |

---

## Client MUST — Winlogon input (C-RD-IN-WL-*)

| ID | Rule |
|----|------|
| **C-RD-IN-WL-1** | While stream is `winlogon_mode` / `desktop=Winlogon`, all `key` / `type_text` / pointer events MUST inject into that **same** interactive session + Winlogon (or Default after unlock) helper as capture — not Session 0. |
| **C-RD-IN-WL-2** | Protocol-2 inputs on Winlogon stream emit `input_ack` with `success` reflecting helper inject, not merely WS receipt. |
| **C-RD-IN-WL-3** | Stream meta/stats MUST expose `inputs_applied` (and optional `last_input_event`) so cloud/lab can verify without guessing. |
| **C-RD-IN-WL-4** | After SAS succeeds, subsequent Tab/Enter/`type_text` reach the credential UI without requiring a second Start. |

---

## Acceptance (lab)

Host: lock/logon UI live (C-RD-S0 green). Agent ≥ target.

- [ ] CAD → within 2s stream shows SAS/security options **or** password field (not unchanged CAD tip) — **open after 4.9.85**  
- [x] Three consecutive CAD while stuck on tip → result NOT all `success:true` with only `SendSAS(0) called` — **4.9.85 → `SAS_NO_EFFECT`**  
- [ ] `SOFTWARE_SAS_DISABLED` when policy blocks services — **lab still reports `software_sas_generation:null`**  
- [ ] After real SAS: type password chars via toolbar/`type_text` appear in field (or masked)  
- [x] Meta `inputs_applied` increases on key/click during Winlogon stream — **4.9.85**  
- [ ] Esc dismisses SAS overlay when applicable  

### Residual for **≥4.9.86** (must-fix)

| ID | Gap on 4.9.85 |
|----|----------------|
| **C-RD-CAD-1** | Helper `SendSAS(FALSE)` still no visible SAS |
| **C-RD-CAD-3** | Report real `software_sas_generation` (0/1/2/3); auto-enable Services/Both when agent owns policy or fail `SOFTWARE_SAS_DISABLED` |
| **C-RD-CAD-4/5** | `ui_before`/`ui_after` = `unknown` — improve effect detect (OCR optional; prefer named-desktop / LogonUI state) so `SAS_NO_EFFECT` is trustworthy when tip still visible |

---

## Operator handoff (client chat)

> Contract **1.4.53** residual. **4.9.85** honesty ✅ (`SAS_NO_EFFECT`, `inputs_applied`). Real SAS UI still ❌ on Derin-Web: helper path, `software_sas_generation:null`, `ui_*=unknown`. Ship **≥4.9.86**: effective console SAS + policy value in result + reliable UI effect detect. Docs: `agent/remote-cad-winlogon.md`.
