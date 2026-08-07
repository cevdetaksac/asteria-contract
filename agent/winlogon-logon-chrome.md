# Winlogon / Logon chrome pixels (not solid blue)

> **Contract VERSION:** **1.4.55**  
> Status: **Normative (client P0)**  
> Min client to close: **≥ 4.9.87**  
> Related: [`winlogon-session0-capture.md`](./winlogon-session0-capture.md) ·
> [`remote-cad-winlogon.md`](./remote-cad-winlogon.md) ·
> [`../cloud/remote-console-parity.md`](../cloud/remote-console-parity.md)

## Lab evidence (2026-08-07 — Derin-Web / **4.9.86**)

| Claim (agent/meta) | Observed JPEG/WS frame |
|--------------------|------------------------|
| `streaming:true`, `winlogon_mode:true`, `capture_method=persistent-winlogon-helper:raw` | Solid blue fill |
| CAD `ui_*=sas_ui`, `effect:true`, `software_sas_generation:1` | Same frame: `avg≈(0–1, ~88–91, ~156)`, **`variance≈0`**, **`brightRatio=0`** (no clock / CAD tip / option text) |

Earlier **4.9.84** lab briefly showed full lock wallpaper + “Kilidi açmak için Ctrl+Alt+Delete…”.  
After CAD / 4.9.86 path, operators only see **solid blue** — not a usable Logon/SAS UI.

**Verdict:** Meta CAD honesty (1.4.54) ≠ usable console pixels. Flat-color Winlogon capture is a **P0 capture FAIL**, even if `black_frame=false` and dimensions are 1024×768.

---

## Client MUST (C-RD-CHROME-*)

| ID | Rule |
|----|------|
| **C-RD-CHROME-1** | Logon Start MUST deliver **recognizable LogonUI / lock chrome** within ≤3s: wallpaper *or* dimmed lock with clock/date **and/or** the CAD tip string, **or** SAS security options / password field with visible glyphs. Solid primary-color fill alone is **not** success. |
| **C-RD-CHROME-2** | Treat **near-zero luminance variance** frames (flat blue/black/grey ≥95% area, no bright glyphs) as capture failure: `capture_method` must not stay “healthy raw”; emit `winlogon_capture_flat` / `gdi+flat` (or reuse `winlogon_capture_black` with `flat_frame:true`) and `streaming` honesty accordingly. |
| **C-RD-CHROME-3** | Do **not** classify flat blue as `ui=sas_ui` for CAD effect. `sas_ui` requires detectable SAS chrome (option rows / password box / tip cleared *with* new UI). Flat fill → `ui=other`/`unknown` or `SAS_NO_EFFECT` if tip/chrome missing. |
| **C-RD-CHROME-4** | After CAD, if secure desktop switches, helper MUST recapture **that** desktop’s real pixels (LogonUI/SAS), not a stale solid secondary buffer. |
| **C-RD-CHROME-5** | Hello/meta MAY include `frame_variance` or `chrome_detected:bool` for lab; cloud may surface degraded banner on flat frames (optional). |

---

## Cloud (optional, this drop)

| ID | Rule |
|----|------|
| **CL-RD-CHROME-1** | Viewer: if live frames are flat (variance≈0) while `winlogon_mode`, show degraded “Logon pixels missing (flat/blue)” — not silent “Canlı”. |

---

## Acceptance

Host: Logon row Connect on console lock/logon.

- [ ] ≤3s: visible lock/logon **or** SAS/password chrome (not solid blue)  
- [ ] Sampled frame `brightRatio>0` or variance clearly non-zero (glyphs/wallpaper)  
- [ ] Flat blue → explicit capture error / degraded, not `success` health  
- [ ] CAD: `ui=sas_ui` only with non-flat SAS chrome  

---

## Operator handoff

> Contract **1.4.55** / `agent/winlogon-logon-chrome.md`. Derin-Web **4.9.86**: CAD meta OK but stream is **solid blue** (`variance=0`). Not a real Logon UI. Fix capture of Winlogon/LogonUI/SAS chrome (C-RD-CHROME-1…5). Target **≥4.9.87**.
