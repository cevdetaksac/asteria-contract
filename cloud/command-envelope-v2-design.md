# Command Envelope v2 + hardware-backed identity — design archive

> Contract: **1.4.37** (schema promoted)  
> **Normative schema SoT:** [`../api/12-command-envelope-v2.md`](../api/12-command-envelope-v2.md)  
> Status: design history + TPM notes; **production wire emit still OFF**  
> Current production remains v1 HMAC + client soft-allow/observe.  
> Client hello may advertise `caps.command_envelope_v2: "off"|"observe"` only
> (see `api/03-control-websocket.md`). Operator key distribution draft:
> [`operator-keyset-design.md`](./operator-keyset-design.md).

## Goals

- operator authorization remains verifiable if cloud routing is compromised;
- bind command to tenant, device, params, expiry, policy and approval set;
- reject replay while preserving idempotent retry/result behavior;
- allow future device payload encryption independently from authorization;
- support TPM-backed device proof and controlled re-enrollment.

## Locked decisions → see api/12

Serialization (JCS), Ed25519, custody, approvals, replay, key lifecycle, and
the canonical envelope JSON are **normative** in
[`../api/12-command-envelope-v2.md`](../api/12-command-envelope-v2.md).
Do not fork decisions here.

## TPM device identity candidate (still design)

- Device generates non-exportable signing key; cloud stores public key,
  attestation state and enrollment generation.
- Proof-of-possession binds `device_id`, tenant, nonce and current enrollment
  generation.
- MachineGuid remains compatibility identity during pilot; TPM absence is
  `unsupported`, not failure.
- Re-enrollment requires explicit owner approval, records old/new key ids and
  invalidates cloned enrollment generations.
- No irreversible hardware lock. Replacement motherboard/TPM clear has a
  documented recovery path.

## Rollout gates (unchanged intent)

1. ~~Promote exact serialization, algorithm, errors and test vectors into api~~
   → **done in 1.4.37** (`api/12`).
2. Implement cloud dual-read/dual-write; v1 remains available.
3. Client advertises capability and verifies v2 in observe mode.
4. Dashboard shows key/approval/replay/rollback status.
5. Internal pilot, downgrade and compromised-cloud tests pass.
6. Enforcement/floor change ships in a separate explicit contract release.

**No code may emit a production `version:2` command before emit gates in
`api/12` §Emit gates are all green.**
