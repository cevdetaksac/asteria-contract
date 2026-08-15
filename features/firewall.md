# Firewall — single contract

> **SoT** **≥ 1.4.65** · Client inventory **≥ 4.9.40** · MMC parity **≥ 4.9.41** · brand **≥ 4.9.33**
> Shared AR-BLOCK wire: [`../api/06-firewall-blocks.md`](../api/06-firewall-blocks.md)

Two families:

| Prefix | Meaning |
|--------|---------|
| **AR-BLOCK-*** | Auto-response / operator IP blocks (`06`) |
| **AR-INTEL-*** | Threat-intel IoC blocks ([`threat-intel.md`](./threat-intel.md)) |

Brand: never write new `HP-*`. Client ≥4.9.33 rewrites leftovers.

Dashboard: Asteria inventory (`list_firewall`) + Windows MMC parity (all rules, profiles).
`firewall_set_profile` is confirm-gated. `firewall_rule` enable/disable is not a blanket confirm.
`block-removed` ACK must include **block_ids and ips** or dashboard stays “Kaldırılıyor?”.
