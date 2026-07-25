# Command Envelope v2 — normative schema (observe-only wire)

> **Contract VERSION:** **1.4.37**  
> Status: **Normative schema** · **Wire emit = OFF** · **Verify enforce = OFF**  
> Design history: [`../cloud/command-envelope-v2-design.md`](../cloud/command-envelope-v2-design.md)  
> Keys: [`../cloud/operator-keyset-design.md`](../cloud/operator-keyset-design.md)  
> Promote criteria: [`../cloud/PROMOTION_GATES.md`](../cloud/PROMOTION_GATES.md)  
> Hello cap: `caps.command_envelope_v2` = `"off"` \| `"observe"` only (`api/03`)

Production remains **v1 HMAC** (`asteria-chp-v1`). This document locks the v2
shape and crypto so client/cloud can implement dual-read/observe without
emitting `version:2` in production until §Emit gates pass.

---

## Locked decisions (was design gate)

| # | Topic | Decision |
|---|--------|----------|
| 1 | Serialization | **RFC 8785 JCS** (canonical JSON) over the envelope object **excluding** `signature` |
| 2 | Algorithm | **Ed25519** — `signature` = base64url(raw 64-byte sig) over JCS bytes (UTF-8) |
| 3 | Signer custody | Operator private key **not** held by cloud routing. Preferred: WebAuthn / platform authenticator or operator HSM; cloud stores **public** keys only |
| 4 | Approvals | High-impact (destructive catalog): ≥1 approval signature covering envelope hash; `network_restore` / isolate-class: ≥2 distinct `operator_id`s unless break-glass policy |
| 5 | Replay | Durable store of `(tenant_id, command_id)` + `nonce`; window **24h**; duplicate delivery returns same terminal result, never re-executes |
| 6 | Key lifecycle | Per [`operator-keyset-design.md`](../cloud/operator-keyset-design.md): register, overlap rotation, revoke, break-glass |
| 7 | Confidentiality | **Out of band** for v2 authz — optional HPKE later; routing metadata stays visible |

---

## Canonical envelope

```json
{
  "version": 2,
  "tenant_id": "tenant-uuid",
  "device_id": "device-uuid",
  "command_id": "command-uuid",
  "command_type": "network_restore",
  "params_hash": "sha256:<lowercase-hex>",
  "issued_at": "2026-07-22T00:00:00.000000Z",
  "expires_at": "2026-07-22T00:05:00.000000Z",
  "nonce": "<128-bit base64url>",
  "operator_id": "operator-uuid",
  "key_id": "operator-key-id",
  "policy_version": "policy-uuid",
  "approvals": [
    {
      "operator_id": "approver-uuid",
      "key_id": "approver-key-id",
      "signed_at": "2026-07-22T00:00:01.000000Z",
      "signature": "<base64url>"
    }
  ],
  "signature": "<base64url>"
}
```

### Field rules

| Field | Rule |
|-------|------|
| `version` | Literal `2` |
| `params_hash` | `sha256:` + lowercase hex of JCS(`params`) or empty-object JCS if no params |
| `issued_at` / `expires_at` | UTC ISO-8601 with `Z`; `expires_at` − `issued_at` ≤ 15 minutes for mutating commands |
| `nonce` | 128-bit cryptographically random, base64url, unique per command_id |
| `approvals[].signature` | Ed25519 over JCS of approval body **without** its `signature`, bound to parent envelope hash (see vectors) |
| `signature` | Issuer Ed25519 over JCS of full envelope **without** `signature` |

### Error codes (observe + future enforce)

| Code | When |
|------|------|
| `envelope_version_unsupported` | version ≠ 2 when v2 path selected |
| `envelope_expired` | now > expires_at |
| `envelope_replay` | command_id/nonce seen |
| `envelope_sig_invalid` | bad issuer or approval sig |
| `envelope_key_unknown` / `envelope_key_revoked` | key_id not trusted |
| `envelope_params_mismatch` | params_hash ≠ local params |
| `envelope_policy_mismatch` | policy_version stale / unknown |

---

## Test vector (fixture)

Fixed Ed25519 seed for CI only (never production keys):

```text
seed_hex = 000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f
```

Implementations MUST publish their derived public key + one golden
`signature` for the minimal envelope below (empty `approvals`) in client/cloud
unit tests. Exact golden bytes live in repo test fixtures once dual-read lands;
schema compliance is mandatory now.

Minimal signable object (illustrative — regenerate goldens in code):

```json
{
  "version": 2,
  "tenant_id": "00000000-0000-4000-8000-000000000001",
  "device_id": "00000000-0000-4000-8000-000000000002",
  "command_id": "00000000-0000-4000-8000-000000000003",
  "command_type": "ping",
  "params_hash": "sha256:44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a",
  "issued_at": "2026-07-22T00:00:00.000000Z",
  "expires_at": "2026-07-22T00:05:00.000000Z",
  "nonce": "AAAAAAAAAAAAAAAAAAAAAA",
  "operator_id": "00000000-0000-4000-8000-000000000004",
  "key_id": "test-key-1",
  "policy_version": "test-policy-1",
  "approvals": []
}
```

(`params_hash` above = sha256 of JCS `{}` — verify in fixture generator.)

---

## Client behavior now

| Mode | Behavior |
|------|----------|
| `caps.command_envelope_v2=off` | Ignore v2 fields |
| `observe` | May parse/verify and **log** mismatches; **must not** hard-fail v1 commands; `verify_enabled` stays false for production enforce |
| Future `enforce` | Only after §Emit gates + separate VERSION |

---

## Emit gates (cloud MUST NOT emit `version:2` until ALL true)

1. This schema + errors + fixture goldens in CI (cloud + client).  
2. Dual-read/dual-write with v1 still authoritative.  
3. Dashboard key/approval/replay status visible.  
4. Internal pilot + downgrade + compromised-routing drill pass.  
5. Explicit contract VERSION announcing emit; later VERSION for enforce.

See also [`../cloud/ZERO_TRUST_STATUS.md`](../cloud/ZERO_TRUST_STATUS.md).
