# Service port relocate — single contract

> **SoT** **≥ 1.4.61**. Pointers: [`../agent/service-port-relocate.md`](../agent/service-port-relocate.md),
> [`../agent/service-port-relocate-client.md`](../agent/service-port-relocate-client.md)

One-click real service (RDP/TermService) → safe 4XXXX port. Never 53389 / 9XXXX.
`relocate_service` confirm-gated. Bidirectional sync + GUI relocate-report.
Golden rollback if listen fails (anti-brick).
