# Cloud backlog — open owner: cloud (contract 1.4.48)

> Client **4.9.48** ships RD `stream_progress` (C-RD-PROG). Cloud relay already live ≥1.4.38.
> Anti-brick P0 cloud rows (CL-BRICK-3 / STATUS / UNLINK-MAIL) closed in **1.4.48**.
> Client claim gate + unlink OTP GUI ≥**4.9.75**.
> Do not flip fleet canary masters / silent-hours auto until remaining ops gates are green.

## P0 — anti-brick / account

| ID | Work | Status |
|----|------|--------|
| **CL-BRICK-5** | Admin-class disable → instant undo **email + one-time key** → queued `enable_account` | **shipped** (hooks + redeem ≥1.4.34) |
| **CL-BRICK-STATUS** | `GET /api/agent/account-status` → **`undo_mail_path: bool`** from real mailer/key readiness | **shipped ≥1.4.48** |
| **CL-BRICK-3** | Dashboard authenticated + agent token → idempotent auto-link; foreign link no-steal | **shipped ≥1.4.48** |
| **CL-UNLINK-MAIL** | `POST /api/agent/unlink-account/request` + `confirm_code` on unlink (P0d) | **shipped ≥1.4.48** · client ≥**4.9.75** |
| **CL-BRICK-DEFAULTS** | New threat_config / migrate: silent_hours* auto flags **OFF** | C-BRICK-2 |

## P0 — fleet automation (already wired; ops + cloud env)

| ID | Work | Contract |
|----|------|----------|
| **CL-CANARY** | Keep `ASTERIA_FLEET_CANARY_*` masters default **0**; emit `fleet_rollout{}` on threats/config | [`cloud/FLEET_CANARY.md`](./FLEET_CANARY.md) |
| **CL-CANARY-DOC** | Mark cloud acceptance `[x]` when prod emit verified; client unit already green ≥4.9.45 | same |

## P1 — promote / observe (do not rush)

| ID | Work | Contract |
|----|------|----------|
| **CL-ENV-V2** | Dual-read envelope v2 observe; **no** production `version:2` emit until `api/12` gates | [`api/12-command-envelope-v2.md`](../api/12-command-envelope-v2.md) |
| **CL-OFFLINE** | Offline urgent queue one-host pilot + canary `offline_urgent_queue` ≥7d ≤5% | [`api/10-offline-urgent-queue.md`](../api/10-offline-urgent-queue.md) · [`PROMOTION_GATES.md`](./PROMOTION_GATES.md) |
| **CL-SUNSET** | Dual-brand host / legacy HMAC sunset ops toward **2026-10-01** | [`PRODUCT_BRANDING.md`](./PRODUCT_BRANDING.md) |

## Lab (not cloud code) — client IR

| ID | Work | Contract |
|----|------|----------|
| **LAB-RD-P0** | Lock console → non-black Winlogon ≤3s; UDP block → ICE failed + JPEG ≤2s | [`agent/remote-desktop-p0.md`](../agent/remote-desktop-p0.md) — client code ≥4.9.45 |
| **LAB-RD-PROG** | Bağlan → Queue→Agent(`running`) ≤2s; mid-start kill → `failed`; subtitle without HTTP poll | [`agent/remote-stream-progress.md`](../agent/remote-stream-progress.md) — client ≥**4.9.48**, cloud relay ≥1.4.39 |

## Explicitly not cloud this sprint

- Envelope **enforce** / operator key custody
- `isolate_host` full path (client still P2 stub)
- Removing legacy `yesnext-*` verify before sunset date
