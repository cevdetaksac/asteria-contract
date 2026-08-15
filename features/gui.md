# Control Center GUI — single contract

> **SoT** **≥ 1.4.62**. Pointers: [`../agent/gui-control-center.md`](../agent/gui-control-center.md),
> [`../agent/gui-webview-bridge.md`](../agent/gui-webview-bridge.md)

GUI is frontend-only. Protection / whitelist / ransomware / webhook state comes
from **motor STATUS / IPC**, never in-process engines (`None` in tray). Dual tray
forbidden. WebView2: installer ships Standalone; do not nag when failure is unrelated.
