# Network Adapter Admin — Windows-like NIC control + golden watchdog

> **Behavior SoT:** [`../features/network-adapter.md`](../features/network-adapter.md) (**≥ 1.4.62**). Appendix below.

> **Contract VERSION:** **1.4.42**  
> Status: **Normative (client ≥ 4.9.42)**  
> Related: [`network-guard.md`](./network-guard.md) ·
> [`anti-brick-critical-actions.md`](./anti-brick-critical-actions.md) ·
> [`system-recovery.md`](./system-recovery.md)

## Goal

Give operators a dashboard surface close to Windows **Network Connections /
Adapter properties**:

| Windows action | Asteria |
|----------------|---------|
| Enable adapter | `network_adapter_apply` `op=enable` |
| Disable adapter | existing `network_disable_adapter` **or** `op=disable` |
| DHCP ↔ static IPv4 + prefix + gateway | `op=set_ipv4` |
| DNS servers | `op=set_dns` |
| Combined IPv4+DNS | `op=set_config` |

**Hard safety:** every intentional mutate that can break reachability MUST run a
**local watchdog**. If internet (or gateway, see below) does not recover within
**5–15 s** (default **10**), the client **automatically restores that adapter
(and related ipv4/dns) from the signed golden baseline** — even if cloud is
unreachable.

This is **not** a replacement for Network Guard’s subtractive `auto_restore_network`
(attack / drift defense). It is an **operator-armed apply** with deadman rollback.

Cloud page (target): same `/dashboard/server/network` — **Adapters** tab /
row actions. Deep-link unchanged:
`https://asteria.run/dashboard/server/network?token={TOKEN}`

---

## Why not “just use maintenance + manual edit”?

Today’s safe path is: maintenance → change on host → snapshot. That works when
someone is at the console. Remote dashboard edits need **client-local** rollback
because a bad static IP / DNS / disable can cut the control channel before any
cloud `network_restore` arrives.

---

## Prerequisites

| ID | Rule |
|----|------|
| **C-NAD-PRE-1** | A **verified** golden baseline MUST exist (`list_network_baseline.verified=true`) before any apply. Else refuse `error=NO_GOLDEN` |
| **C-NAD-PRE-2** | Golden connectivity at capture should have been healthy; if last known golden snapshot had `internet_ok=false`, refuse `error=GOLDEN_UNHEALTHY` (operator must fix golden first via console / maintenance) |
| **C-NAD-PRE-3** | Cloud `confirm:true` required for all mutating ops (same class as `network_restore`) |
| **C-NAD-PRE-4** | While an apply watchdog is armed, **pause** Network Guard `auto_restore_network` for that interface only (or globally for ≤ `watchdog_sec + 5`) so Guard does not fight the intentional change mid-flight |

---

## Command: `network_adapter_apply` (destructive confirm)

```json
{
  "adapter": "Ethernet",
  "op": "set_config",
  "ipv4": {
    "dhcp": false,
    "address": "192.168.1.50",
    "prefix_length": 24,
    "gateway": "192.168.1.1"
  },
  "dns": ["1.1.1.1", "8.8.8.8"],
  "watchdog_sec": 10,
  "on_fail": "restore_golden",
  "on_success": "keep",
  "probe": "internet"
}
```

### Ops

| `op` | Body | Notes |
|------|------|-------|
| `enable` | `adapter` | Bring interface up |
| `disable` | `adapter` | Same effect as `network_disable_adapter`; prefer one path |
| `set_ipv4` | `ipv4` | DHCP or static |
| `set_dns` | `dns[]` | Empty / omit = leave unchanged; `dns: ["dhcp"]` or `"dns_from_dhcp": true` = DHCP DNS |
| `set_config` | `ipv4` + optional `dns` | Atomic IPv4+DNS |

### Watchdog params

| Field | Default | Rule |
|-------|---------|------|
| `watchdog_sec` | **10** | Clamp **5..15** |
| `on_fail` | `restore_golden` | Only allowed value in v1 |
| `on_success` | `keep` | `keep` = live may drift from golden until operator Accept; `accept_surface` = promote live→golden **only if** probe passed |
| `probe` | `internet` | `internet` = `connectivity.internet_ok`; `gateway` = gateway_ok only; `internet_and_gateway` = both |

### Result

```json
{
  "success": true,
  "data": {
    "adapter": "Ethernet",
    "op": "set_config",
    "applied": true,
    "rolled_back": false,
    "watchdog_sec": 10,
    "probe": {
      "mode": "internet",
      "ok": true,
      "internet_ok": true,
      "dns_ok": true,
      "gateway_ok": true,
      "elapsed_ms": 1840
    },
    "on_success": "keep",
    "golden_version": 12,
    "live_adapter": {
      "name": "Ethernet", "state": "up", "ipv4": "192.168.1.50",
      "gateway": "192.168.1.1", "dns": ["1.1.1.1", "8.8.8.8"],
      "dhcp": false, "prefix_length": 24
    }
  }
}
```

On fail + rollback:

```json
{
  "success": false,
  "error": "WATCHDOG_ROLLBACK",
  "data": {
    "applied": true,
    "rolled_back": true,
    "probe": { "ok": false, "internet_ok": false, "elapsed_ms": 10050 },
    "restore_actions": ["ipv4:Ethernet", "dns:Ethernet"]
  }
}
```

| ID | Rule |
|----|------|
| **C-NAD-1** | Apply → probe loop until ok or `watchdog_sec` → on fail **local** golden restore for that adapter’s `adapter`+`ipv4`+`dns` targets (not whole-host firewall unless those were changed) |
| **C-NAD-2** | Watchdog MUST run **on the agent** (not cloud). Cloud may never see the failure if the link dies |
| **C-NAD-3** | After successful rollback, emit alert `network_adapter_apply_rolled_back` (warning, not under_attack) |
| **C-NAD-4** | After success with `on_success=keep`, optional soft inform / drift until Accept — do **not** auto-poison golden |
| **C-NAD-5** | After success with `on_success=accept_surface`, write new golden (= `network_accept_surface`) |
| **C-NAD-6** | `disable` / `enable` also use watchdog when the change can cut management path (disable of an up adapter that currently carries default route → always watchdog; enable of secondary → probe optional but recommended) |
| **C-NAD-7** | Refuse `disable` of the **last** adapter that has a default route / was golden `up` with gateway → `error=LAST_MGMT_ADAPTER` (anti-brick). Operator break-glass: console maintenance or physical access |
| **C-NAD-8** | GPO / permission failures → `error=ACCESS_DENIED` / `GPO_LOCKED`; do not claim success |
| **C-NAD-9** | IPv6 / advanced metrics / VLAN / bonding = **out of scope** v1 |

---

## Compatibility with existing commands

| Existing | Keep? | Note |
|----------|-------|------|
| `network_disable_adapter` | yes | Still valid; dashboard may call `network_adapter_apply` `op=disable` instead |
| `network_restore` | yes | Manual / Guard path; watchdog uses same restore engine scoped to adapter |
| `network_maintenance_*` | yes | Preferred for long console work; apply is for **remote one-shot** edits |
| `auto_restore_network` | yes | Remains attack/drift defense; paused only during armed apply window |

---

## Client algorithm (normative)

```
assert golden verified
pause auto_restore_network (scoped)
snapshot pre_state (for diagnostics)
apply op via Windows (netsh / CIM / IPHelper)
deadline = now + watchdog_sec
loop:
  probe connectivity
  if probe_ok: break
  sleep ~500ms
if not probe_ok:
  restore golden targets [adapter, ipv4, dns] for this name
  resume auto_restore_network
  return WATCHDOG_ROLLBACK
if on_success == accept_surface:
  network_accept_surface / snapshot
resume auto_restore_network
return success
```

Probe definition MUST match Network Guard’s `connectivity` probes (same hosts /
timeouts where possible) so “internet_ok” means the same thing everywhere.

---

## Cloud / dashboard (ship with or after client)

1. Network page → **Adapters** section: live table with Enable / Disable / Edit IPv4·DNS  
2. Edit modal → DHCP toggle, address, prefix, gateway, DNS1/DNS2  
3. Sticky confirm → `network_adapter_apply` with `watchdog_sec=10`, `on_success=keep`  
4. Toast on `WATCHDOG_ROLLBACK`: “Değişiklik interneti kesdi — golden’a dönüldü”  
5. Optional checkbox: “Başarılıysa yeni golden kabul et” → `on_success=accept_surface`

Cloud accepts the command type in `VALID_COMMAND_TYPES` + destructive confirm gate
starting **1.4.42** even before UI lands.

---

## Acceptance

- [ ] Lab: set bad static IP → within ≤15s live returns to golden; agent still online  
- [ ] Lab: set good static IP → stays; `keep` leaves drift until Accept  
- [ ] Lab: `accept_surface` on success → golden matches new IP  
- [ ] Lab: disable last mgmt adapter → refused `LAST_MGMT_ADAPTER`  
- [ ] Lab: Guard `auto_restore_network` does not fight mid-apply  
- [ ] Old client: command fails/unknown; existing Network Guard UI unchanged  

---

## Min client

**≥ 4.9.42** (or next train closing C-NAD-*).  
Depends on Network Guard baseline engine ≥ **4.9.12**.
