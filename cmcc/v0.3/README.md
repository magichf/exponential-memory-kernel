# CMCC v0.3 — immutable event model (DRAFT)

**Status:** Draft for independent review. CMCC v0.2 remains the frozen operational baseline; this draft does not replace it.

## Design goals

1. Separate message identity (`message_id`) from event identity (`event_id`).
2. Treat each event as append-only. Never mutate an earlier event to represent a later lifecycle state.
3. Scope every event to a `conversation_id` and a `channel_id`.
4. Include nonce and expiry fields for replay-control inputs, while acknowledging that enforcement requires a trusted receiver/registry.
5. Separate a read receipt from cryptographic identity authentication.
6. Keep Notion and GitHub as independent transports with explicit cross-channel references.

## Event types

- `created`: a message is authored.
- `read_receipt`: a receiver asserts it has read a referenced event.
- `delivered`: a trusted coordinator records delivery after validating a receipt.
- `replied`: a new message responds to an earlier message.
- `acked`: the original sender acknowledges a reply or event.
- `expired`: a receiver/coordinator records that the event is past its validity window.
- `closed`: a coordinator closes the conversation segment.

A `read_receipt` is an assertion, not proof that a particular model generated it. `auth_mode=signed` is meaningful only when the signature verifies against a trusted, previously registered public key. `trusted_agent` requires a separately documented trust boundary.

## State derivation

State is computed from a validated event stream, not stored as mutable truth. A reducer must:
- reject duplicate `event_id`;
- reject reused (actor, channel, nonce) tuples within the replay-retention window;
- validate parent/subject references and conversation/channel scope;
- reject events received after `expires_at`, according to receiver policy;
- verify signatures before assigning a signed identity;
- handle out-of-order arrivals deterministically and preserve the raw append-only log.

The JSON Schema checks shape only. It cannot enforce uniqueness, reference existence, nonce consumption, expiry at receipt time, signature validity, authorization, or atomic append.

## Canonicalization and signatures

Before enabling signatures, specify a canonical serialization (recommended: RFC 8785 JSON Canonicalization Scheme), UTF-8 handling, hash algorithm, signature input domain separation, key registration/rotation/revocation, and clock-skew policy. Do not sign an arbitrary pretty-printed JSON string. A checksum detects accidental content mismatch; it does not authenticate a sender.

## Current implementation boundary

This is a schema and architecture draft stored in GitHub. No nonce registry, signature verifier, polling worker, webhook, relay, or automatic model wake-up has been implemented. GitHub/Notion writes are manual tool-mediated operations at this stage.
