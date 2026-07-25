# Zero Trust — durum (1.4.37)

> Envelope v2 **şema normatif** (`api/12`); **wire emit / enforce hâlâ kapalı**.  
> Bu dosya cloud’a “şimdi ne yapmalı / ne yapmamalı” netliği verir.

## Gerçek durum

| Paket | Statü | Dosya |
|-------|--------|--------|
| v1 HMAC command signing | Normative (soft-allow missing sig) | `api/03-control-websocket.md` |
| v1 coverage metrics | Live on resilience status | `GET …/security-resilience/status` |
| `caps.command_envelope_v2` | Observe only (`off`\|`observe`) | `api/03` |
| ZT-601 envelope v2 **schema** | **Normative (observe-only)** | `api/12-command-envelope-v2.md` |
| ZT-601 wire emit / enforce | **Gated** | `api/12` §Emit · `PROMOTION_GATES.md` |
| ZT-602/603 operator keys | Design + cloud observe stub | `cloud/operator-keyset-design.md` |
| TPM PoP / enrollment | Design-only | design archive + `device_identity` observe |
| Fleet automation canary | Normative | `cloud/FLEET_CANARY.md` |
| Observe→enforce criteria | Normative | `cloud/PROMOTION_GATES.md` |

Client scaffold may parse/verify in observe; production `verify_enabled` /
hard-fail on v2 remains false until emit+enforce VERSIONs.

## Cloud’un şimdi yapması (1.4.37)

1. **Yapma:** fleet’e `version:2` asymmetric envelope emit / enforce  
2. **Yap:** v1 HMAC coverage ölçümü — `command_signing.fleet` + `enforce_ready`  
3. **Yap:** `fleet_rollout` canary (auto flag clear) — production default percent=0  
4. **Yap (observe):** `GET /api/agent/operator-keys` public-only stub (`verify_enabled:false`)  
5. **Sonra:** dual-read/write + goldens CI → pilot → ayrı VERSION emit → ayrı VERSION enforce

## “Hacklenirseniz biz de hackleniriz”

Bugün savunma: Bearer token + v1 HMAC soft-allow + confirm-gated mutates +
Network Guard / System Recovery + canary-gated autos.  
Asimetrik ZT bunu güçlendirir ama **tek başına SaaS kapısı değil**.

Detay: [`SECURITY_RESILIENCE_VNEXT.md`](../SECURITY_RESILIENCE_VNEXT.md) ·
[`PROMOTION_GATES.md`](./PROMOTION_GATES.md).
