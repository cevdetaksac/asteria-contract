# Persistence / Guardian — single contract

> **SoT** **≥ 1.4.62**. Pointer: [`../agent/persistence-and-tamper.md`](../agent/persistence-and-tamper.md)

AsteriaGuardian + watchdog watch the **SYSTEM motor**, not the GUI tray.
Tamper / unexpected_exit must not fire on planned installer handoff
(`update_in_progress.lock` until new motor is ready). Operator PIN stop is signed.
