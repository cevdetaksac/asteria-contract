# Winlogon / Logon chrome pixels (not solid blue)

> **Contract VERSION:** **1.4.56**  
> Status: **Normative (client P0)**  
> Min client (chrome honesty): **≥ 4.9.87**  
> Min client (live meta / soft-start lab): **≥ 4.9.91**  
> Related: [`winlogon-session0-capture.md`](./winlogon-session0-capture.md) ·
> [`remote-cad-winlogon.md`](./remote-cad-winlogon.md) ·
> [`../cloud/remote-console-parity.md`](../cloud/remote-console-parity.md) (**C-RD-FOLLOW**, 1.4.58)

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
| **C-RD-CHROME-6** | **≥4.9.91** per-frame `t:meta` MUST carry (additive): `capture_method`, `chrome_detected`, `flat_frame`, `frame_variance`, `bright_ratio`, `logonui_hwnd_count`, `desktop`, `inputs_applied`, `last_input_event`. Do not freeze hello/`remote_stream_start` snapshot after settle. |
| **C-RD-CHROME-7** | Soft-start (**≥4.9.90**): `success=true` + early `gdi+flat` / `chrome_detected=false` / `hwnd=0` is **not** terminal. `winlogon_capture_flat` only on real error or `stream_progress.phase=failed`. |
| **C-RD-CHROME-8** | `stream_progress.phase=degraded` = “LogonUI chrome bekleniyor”. Not `failed`. Do not emit `live`/`connected` while `+flat`/`+black`. |

---

## Cloud (shipped 1.4.56)

| ID | Rule |
|----|------|
| **CL-RD-CHROME-1** | Viewer: black-only frames still degraded; **do not** treat soft-start `gdi+flat` start JSON as connect fail. |
| **CL-RD-CHROME-2** | `GET /api/remote/status` + viewer live stats MUST prefer **last `t:meta` / frame meta** for settle fields — not `remote_stream_start` snapshot. |
| **CL-RD-CHROME-3** | Relay `stream_progress.phase=degraded`; dashboard must not map it to fail. |
| **CL-RD-CHROME-4** | Lab proof for input is **`inputs_applied++`** and `last_input_event=type_text` after settle — toast is not proof. |

---

## Lab (≥4.9.91) — do not score start JSON

Wait a few seconds after `streaming=true`, then read **live meta / get_status**:

- `capture_method` **raw** or **PrintWindow** (stale `gdi+flat` is pre-settle)
- `chrome_detected=true`, `frame_variance≫0`, `logonui_hwnd_count≥1`
- after `type_text`: `inputs_applied++`, `last_input_event=type_text`

CAD: `ui_before`/`ui_after` may be Session-0 `unknown`; accept `effect=true` or honest ui/fp **plus** JPEG bullets.

---

## Acceptance

Host: Logon row Connect on console lock/logon.

- [ ] Start `success=true` with early flat is allowed (soft-start)  
- [ ] After settle: live meta method/chrome/variance/hwnd as above  
- [ ] `phase=degraded` ≠ fail  
- [ ] `type_text` → bullets **and** `inputs_applied++`  
- [ ] CAD: `effect=true` or honest helper ui/fp + JPEG chrome  

---

## Operator handoff

> Contract **1.4.58**. Dashboard default = Logon/console. After Enter, client **≥4.9.93** MUST follow Default (C-RD-FOLLOW). Lab **≥4.9.91**: score settle live meta, not start snapshot.
