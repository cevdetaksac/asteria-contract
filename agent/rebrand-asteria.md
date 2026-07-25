# Brand migration — Asteria (single SoT)

> **Contract VERSION:** **1.4.32**  
> **Min client for full identity:** **≥ 4.9.35**  
> Related: firewall migrate [`firewall-brand-migrate.md`](./firewall-brand-migrate.md) (1.4.31 / ≥4.9.33)

This file is the **only** place that documents former YesNext / Cloud Honeypot names.
Everywhere else in cloud + client product surfaces must say **Asteria**.

Former product name: **YesNext Cloud Honeypot**  
Current: **Asteria** · tagline **Deception Cloud** · primary origin **`https://asteria.run`**

---

## Cutover summary (1.4.32)

| Layer | Was (legacy — keep for verify/wipe/failover) | Is now (Asteria) |
|-------|-----------------------------------------------|------------------|
| Public API / WS | `https://honeypot.yesnext.com.tr` | **`https://asteria.run`** (+ nginx alias for old host until fleet migrates) |
| Command HMAC context | `yesnext-chp-v1` | **`asteria-chp-v1`** (cloud **signs** this; cloud **verify** accepts both during cutover) |
| Heartbeat HMAC context | `yesnext-heartbeat-v1` | **`asteria-heartbeat-v1`** (client must emit new; cloud accept both if verify enabled) |
| Firewall rule prefix | `HP-BLOCK-*` / `HP-INTEL-*` / `HONEYPOT*` | **`AR-BLOCK-*` / `AR-INTEL-*`** (wipe still clears old prefixes) |
| Install dir (new) | `Program Files\YesNext\Cloud Honeypot\` | **`Program Files\Asteria\`** |
| Exe (new) | `honeypot-client.exe`, `yesnext-honeypot-client.exe` | **`asteria-client.exe`** (cloud path-trust still accepts legacy names) |
| ProgramData (existing installs) | `%ProgramData%\YesNext\CloudHoneypotClient\` | **Do not silent-move** — keep until dedicated migrate |
| Client GitHub releases | `cevdetaksac/yesnext-cloud-honeypot-client` | **`cevdetaksac/asteria-client`** |
| Dashboard CSS vars/classes | `--yn-*` / `.yn-*` | **`--ast-*` / `.ast-*`** |
| UI / mail / OpenAPI title | Honeypot / YesNext | **Asteria** |

Industry word **honeypot** (decoy service) may appear in marketing/docs as a *category* term; it is **not** the product brand.

---

## P0 — Client must change (breaking if skipped)

### 1) Command signing context → `asteria-chp-v1`

```text
key = SHA256("{token}|{COMPUTERNAME}|asteria-chp-v1")   // raw 32-byte digest
msg = "{command_id}|{command_type}|{issued_at}"
signature = HMAC-SHA256(key, msg).hexdigest()
```

- Cloud **1.4.32+** produces signatures with **`asteria-chp-v1` only**.
- Client **must verify** with `asteria-chp-v1`. Old `yesnext-chp-v1` verify will reject live commands.
- Defense-rules HMAC uses the **same** context family (`asteria-chp-v1`).
- See [`../api/03-control-websocket.md`](../api/03-control-websocket.md).

### 2) Heartbeat signing context → `asteria-heartbeat-v1`

```text
key = SHA256("{token}|{hostname.lower()}|asteria-heartbeat-v1")
msg = "v1|{hostname.lower()}|{status}|{1|0}|{issued_at}"
```

Legacy `yesnext-heartbeat-v1` is documented only for rollback labs. See [`../api/01-auth.md`](../api/01-auth.md).

### 3) API base

| | |
|--|--|
| **Primary** | `https://asteria.run` |
| **WS** | `wss://asteria.run/ws/agent/control` (+ remote WS same host) |
| **Legacy failover** | `https://honeypot.yesnext.com.tr` only if primary unreachable |

### 4) Visible brand

Tray, About, installer, Start Menu, account copy: **Asteria** — never YesNext / Cloud Honeypot as product name.

### 5) Firewall

Wire writes **`AR-*`** only. Wipe/delete must still remove **`HP-*`** + **`HONEYPOT*`**. Details: [`firewall-brand-migrate.md`](./firewall-brand-migrate.md).

---

## P1 — Paths / trust (dual during transition)

Cloud path-trust accepts:

- `Program Files\Asteria\...` + `asteria-client.exe`
- Legacy: `Program Files\YesNext\...`, `honeypot-client.exe`, `yesnext-honeypot-client.exe`, `cloud-honeypot-client.exe`

New installs must use Asteria paths. Do **not** silently move ProgramData.

---

## P2 — Optional / ops

- Contract git repo may still be named `honeypot-contract` (historical remote).
- Cloud backup git remote may still be `honeypot-cloud` until renamed.
- OS safety tooling may still live under `/usr/local/lib/honeypot-safety` with aliases `honeypot-trash` — archives go to `/data/asteria-trash` / `/data/asteria-snapshots`.

---

## Wire names that stay (not brand UI)

These are protocol / taxonomy identifiers — rename only with a dedicated API version:

- `POST /api/honeypot-attack` (path)
- Threat types: `honeypot_credential`, `honeypot_hit`, …
- Flag `wipe_all_honeypot_rules` (alias of wipe_prefixes)

---

## Yasak

1. Cloud/dashboard UI showing YesNext or “Honeypot Dashboard” as brand.
2. Signing with `yesnext-chp-v1` on new cloud builds (1.4.32+).
3. Forgetting `HP-*` in wipe lists.
4. Silent ProgramData migrate.
5. Removing legacy host from nginx before fleet cutover.

---

## Test matrix

| # | Scenario | Expected |
|---|----------|----------|
| 1 | Client ≥4.9.35 verifies `asteria-chp-v1` | commands accepted |
| 2 | Old client still on `yesnext-chp-v1` verify | commands **fail** until update |
| 3 | Fresh install → `asteria.run` | register + WS ok |
| 4 | Legacy host alias | old agent still heartbeats |
| 5 | Path trust YesNext **and** Asteria dirs | self-proof ok |
| 6 | Firewall wipe | removes AR-* and HP-* |

---

## Sürüm

| Contract | Client min | Notes |
|----------|------------|-------|
| 1.4.30 | ≥4.9.32 | Domain + UI Asteria; signing still legacy |
| 1.4.31 | ≥4.9.33 | Firewall AR-* |
| **1.4.32** | **≥4.9.35** | **Signing/heartbeat contexts Asteria; CSS `--ast-*`; full cloud brand scrub** |
