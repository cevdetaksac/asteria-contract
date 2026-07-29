# Dashboard deep-links (`?token=`)

> **Contract:** ≥ **1.4.35** · **Audience:** Windows client GUI / Settings shortcuts + cloud emails  
> **Base:** `https://asteria.run`  
> **Auth note:** Agent JSON/WS API uses `Authorization: Bearer` only — **not** `?token=`.  
> Query `?token=` is **browser deep-link only** (dashboard HTML). See [`api/01-auth.md`](../api/01-auth.md).

Client Settings / tray / “Open in browser” shortcuts must build URLs with the **current agent token** (after rotate: use `new_token` from `POST /api/agent/rotate-token`).

---

## Canonical form

```
https://asteria.run{path}?token={TOKEN}
https://asteria.run{path}?token={TOKEN}&{extra}=…
https://asteria.run{path}?token={TOKEN}#{hash}
```

| Piece | Rule |
|-------|------|
| `{TOKEN}` | Agent UUID from ProgramData / register / rotate |
| `{path}` | Absolute path starting with `/dashboard` (or `/servers` for account fleet) |
| Encode | URL-encode token if unsure (`encodeURIComponent`) |
| Helper (cloud) | `brand.dashboard_url(token, path="/dashboard")` → `{BASE}{path}?token={token}` |

Register / rotate responses already return the home link:

```json
"dashboard": "https://asteria.run/dashboard?token=<uuid>"
```

**Do not** invent hosts (`localhost`, old brand domains). Legacy API alias hosts are failover-only — deep-links for operators stay on **`asteria.run`** ([`agent/rebrand-asteria.md`](../agent/rebrand-asteria.md)).

---

## Page catalog (token-scoped)

`Premium` = plan gate on nav (STD may still open login / upsell; treat as premium UX).

| Path | Purpose | Plan | Suggested client shortcut label |
|------|---------|------|----------------------------------|
| `/dashboard` | Overview / home | All | Open dashboard |
| `/dashboard/attacks` | Attack log | All | Attacks |
| `/dashboard/threats` | Threat Center | Premium | Threats |
| `/dashboard/blocks` | Firewall blocks + rules tabs | Premium | Blocking / IP rules |
| `/dashboard/ports` | Listening ports | All | Ports |
| `/dashboard/server/firewall` | Windows FW parity + Asteria blocks | All† | Firewall |
| `/dashboard/tunnels` | Bait tunnels | Premium | Tunnels |
| `/dashboard/threat-intel` | Cloud IoC intel UI | Premium | Threat intel |
| `/dashboard/remote` | Remote desktop | Premium* | Remote desktop |
| `/dashboard/server/users` | Local users + sessions | All† | Users |
| `/dashboard/server/processes` | Processes | All† | Processes |
| `/dashboard/server/services` | Windows services | All† | Services |
| `/dashboard/server/network` | Network Guard panel | All† | Network |
| `/dashboard/server/recovery` | System recovery | All† | Recovery |
| `/dashboard/settings` | Account / notify / password | All | Settings |
| `/dashboard/logout` | End **server** dashboard session | All | (rarely expose) |

\* Remote desktop floor: client ≥ **4.9.0** ([`api/05-remote-desktop.md`](../api/05-remote-desktop.md)).  
† Server management: target client ≥ **4.9.4** ([`agent/server-management.md`](../agent/server-management.md)).

### Account fleet (no agent token in path — account cookie)

| Path | Purpose |
|------|---------|
| `/servers` | My servers / link / manage |
| `/?login=1` | Account login entry |

Link flow for unlinked agents: copy token → open `/servers` ([`api/02-account.md`](../api/02-account.md) P0–P3).

---

## Optional query parameters

| Page | Param | Meaning |
|------|-------|---------|
| `/dashboard/attacks` | `service=` | Filter bait/service name, or `all` |
| `/dashboard/attacks` | `ip=` | Filter attacker IP |
| `/dashboard/threats` | (filters via UI) | Prefer path + token; clear filter = bare threats URL |
| `/dashboard/remote` | `session_id=` | Jump into existing RD session |
| `/dashboard/remote` | `username=` | Prefer session for user |

Examples:

```
https://asteria.run/dashboard/attacks?token={TOKEN}&service=ssh
https://asteria.run/dashboard/attacks?token={TOKEN}&ip=1.2.3.4
https://asteria.run/dashboard/remote?token={TOKEN}&session_id={SID}&username={USER}
```

---

## Optional hash tabs (`/dashboard/blocks`)

| Hash | Tab |
|------|-----|
| `#tab-active-blocks` | Active blocks (default) |
| `#tab-whitelist` | Whitelist |
| `#tab-auto` | Auto-block settings |
| `#tab-rules` | Manual rules |
| `#tab-schedule` | Schedule / silent hours |
| `#tab-notifications` | Notifications |
| `#tab-advanced` | Advanced |

Examples:

```
https://asteria.run/dashboard/blocks?token={TOKEN}#tab-auto
https://asteria.run/dashboard/blocks?token={TOKEN}#tab-rules
https://asteria.run/dashboard/blocks?token={TOKEN}#tab-advanced
```

---

## Recommended Settings shortcuts (client GUI)

Minimum set for tray / Settings → Links:

| ID | URL template |
|----|----------------|
| `dash_home` | `https://asteria.run/dashboard?token={TOKEN}` |
| `dash_attacks` | `https://asteria.run/dashboard/attacks?token={TOKEN}` |
| `dash_threats` | `https://asteria.run/dashboard/threats?token={TOKEN}` |
| `dash_blocks` | `https://asteria.run/dashboard/blocks?token={TOKEN}` |
| `dash_users` | `https://asteria.run/dashboard/server/users?token={TOKEN}` |
| `dash_remote` | `https://asteria.run/dashboard/remote?token={TOKEN}` |
| `dash_settings` | `https://asteria.run/dashboard/settings?token={TOKEN}` |
| `dash_servers` | `https://asteria.run/servers` *(no token — account session)* |

Open with default OS browser (`shell.open_dashboard` / equivalent — [`agent/gui-webview-bridge.md`](../agent/gui-webview-bridge.md)).

After **token rotate**, rebuild all templates with the new token; stale bookmarks hit login / wrong client.

---

## Auth / session behavior (operators)

1. Browser hits `…?token={TOKEN}` → cloud resolves client by token → may require **account password** if server is account-linked (no separate dash password when linked).
2. Authenticated account + foreign-or-unlinked token may **auto-link** (C-BRICK-3) — [`agent/anti-brick-critical-actions.md`](../agent/anti-brick-critical-actions.md).
3. Token in URL is a **capability**; treat like a secret in screenshots / chat. Prefer copy-from-Settings over pasting into public tickets.

---

## Client checklist

- [ ] Home deep-link matches register/`dashboard` field host (`asteria.run`)
- [ ] Settings shortcuts use table above; no hard-coded legacy host
- [ ] Rotate-token → refresh stored shortcut base token
- [ ] Agent HTTP/WS never sends `?token=` (Bearer only)
- [ ] Optional: Blocks `#tab-*` and Attacks `service`/`ip` filters for advanced shortcuts

---

## Open questions

- None for catalog completeness as of **1.4.35**. New `/dashboard/*` routes → add a row here in the same VERSION bump.
