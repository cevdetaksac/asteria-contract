# Firewall — single contract

> **SoT** **≥ 1.4.61**. Pointers: [`../api/06-firewall-blocks.md`](../api/06-firewall-blocks.md),
> [`../agent/firewall-management.md`](../agent/firewall-management.md),
> [`../agent/firewall-windows-parity.md`](../agent/firewall-windows-parity.md),
> [`../agent/firewall-brand-migrate.md`](../agent/firewall-brand-migrate.md)

Two families:

| Prefix | Meaning |
|--------|---------|
| **AR-BLOCK-*** | Auto-response / operator IP blocks (`06`) |
| **AR-INTEL-*** | Threat-intel IoC blocks ([`threat-intel.md`](./threat-intel.md)) |

Brand: never write new `HP-*`. Client ≥4.9.33 rewrites leftovers.

Dashboard: Asteria inventory (`list_firewall`) + Windows MMC parity (all rules, profiles).
`firewall_set_profile` is confirm-gated. `firewall_rule` enable/disable is not a blanket confirm.
`block-removed` ACK must include **block_ids and ips** or dashboard stays “Kaldırılıyor?”.
