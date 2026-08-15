# Server management — single contract

> **SoT** **≥ 1.4.61**. Pointers: [`../agent/server-management.md`](../agent/server-management.md),
> [`../cloud/SERVER_USER_MANAGEMENT.md`](../cloud/SERVER_USER_MANAGEMENT.md),
> process evidence: [`process-inspect.md`](./process-inspect.md)

Three dashboard pages: users, processes, services. Agent returns **full inventory**
on refresh and executes catalog mutates. No covert shell.

Users: local SAM + WTS; disable not delete; `include_disabled`; C-USER confirm on
create / remote_logon / reboot. Processes: lean health + `inspect_process` (no confirm).
Services: pywin32 SCM; protected services refuse stop.
