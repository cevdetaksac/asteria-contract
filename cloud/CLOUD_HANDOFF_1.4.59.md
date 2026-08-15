# Cloud handoff — contract 1.4.59 (paste to dashboard / API)

Ship these **this week**. Client **4.9.95** names the topology; old
`prefer=winlogon` + omit `session_id` still means **follow**, not lock.

## P0 — Remote Start (Derin-Web black / spawn fail)

**Stop** using one Start shape for every Connect.

| Operator click | Params (required) | Must not |
|----------------|-------------------|----------|
| Default Connect / “Logon · varsayılan” / follow console | `topology: "follow"` — **omit** `session_id`, **omit** `username`. `prefer` optional (`follow` or omit). | `topology: "winlogon"` · `prefer: "winlogon"` as a *forced lock* |
| Logon / Lock **row** (sibling, empty host, CAD to SAS) | `topology: "winlogon"` + `prefer: "winlogon"` + `pre_logon: true` + `desktop: "Winlogon"` — omit username; omit SID unless that row has one | Treat this as Default desktop |
| User/session shortcut | `session_id` + `username` — no winlogon prefer | Auto-pick first Active SID |

Client ≥**4.9.94/95**: omit-SID + live user Default → **no Winlogon helper**.
If dashboard keeps sending lock-shaped Start for “varsayılan”, 4.9.95 will
still spawn Winlogon when `topology=winlogon`.

**C-RD-VIEW-5 (updated):** default Connect = **follow console**, not Winlogon helper.

After Enter: same `stream_id`, no second Start, no “pick administrator”
(C-RD-FOLLOW). Min agent gate: recommend **≥4.9.95** (follow+named topology).

ICE fail: JPEG-WS on the **same** surface ≤2s; do not leave a 4fps nvenc
leftover badge while the player is dead.

## P0 — `inspect_process` (1.4.57 still open on UI)

Command is **not destructive**. Confirm gate: **no**.

- `VALID_COMMAND_TYPES` includes `inspect_process`
- **?** / İncele → queue `{ "pid": N }` **immediately**
- **Forbidden:** “Onayla ve Gönder” / generic destructive confirm modal
- Do not map image name `rundll32.exe` to **Kritik**
- `rundll32 foo.dll,Entry` is normal; http / javascript: / UNC is lolbin
- Older agent → “client ≥4.9.93 gerekli”, not a silent fail

## P1 — Trust / update UI (do not block download)

`trust metadata pending` / null signing fields = **observe**. Must **not**
block `self_update` or show as the reason 0% is stuck.

Fleet on **≤4.9.72**: one **elevated** installer land (chicken-egg). Cloud
cannot patch ACL/helper on the old agent. Do not stack overlapping Update now.

## P1 — Dual installer name

Publish **only** `asteria-client-installer.exe`. Stop requiring
`cloud-client-installer.exe` except for agents **≤4.9.40** if any remain.

## Acceptance (dashboard)

1. Derin-Web, user already at desktop: default Connect → wallpaper/shell,
   `desktop=default`, **no** `SESSION0_HELPER_SPAWN_FAILED`.
2. Empty or locked host: Logon/Lock **row** → logon/lock pixels.
3. Process **?**: inspect modal without confirm; rundll32 Control Panel not Kritik.
