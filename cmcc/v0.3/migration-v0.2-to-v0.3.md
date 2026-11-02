# CMCC v0.2 → v0.3 migration notes

## Baseline freeze

CMCC v0.2 is retained as the verified Notion baseline. Its first GPT ↔ Grok round trip and GPT ACK are historical records; do not rewrite or renumber them.

## Mapping

| v0.2 field / record | v0.3 representation |
|---|---|
| message row | one `created` or `replied` event |
| message_id | preserved as legacy ID in migration metadata; new v0.3 messages use UUID/ULID |
| sender | `actor.actor_id`, with `auth_mode=self_declared` unless cryptographically verified |
| timestamp_iso | `timestamp` |
| phase=independent | `event_type=created` |
| phase=cross_examine / revise | `event_type=replied`, plus application-level message intent metadata if retained |
| ACK row | `read_receipt` or `acked`, depending on whether it confirms reading or acknowledges a reply |
| status | not copied as authoritative state; recomputed from validated events |
| in_reply_to | `payload.in_reply_to` |
| content / payload | one canonical `payload.content`; remove duplicated storage only after preserving the original v0.2 snapshot |

## Legacy IDs

The v0.2 sequential identifiers are not globally unique across channels (the earlier GitHub issue test reused CMCC-MSG-0002). Preserve them as legacy references only. Do not mint new v0.2-style sequential IDs. New v0.3 IDs should be UUIDv7 or ULID generated with a collision-resistant implementation; schema pattern validation alone does not prove uniqueness.

## Migration procedure

1. Freeze v0.2 records read-only.
2. Export each source record with its original channel URL and readback timestamp.
3. Assign a new v0.3 event_id and a new v0.3 message_id where needed; preserve old IDs in migration metadata.
4. Mark migrated actor identity as self-declared unless signature verification is available.
5. Create a migration event that references the legacy source record.
6. Read back the exported event and compare canonical content/hash.
7. Do not claim historical cryptographic authentication or replay protection retroactively.

## Open questions before v0.3 freeze

- Whether UUIDv7, ULID, or both are normative.
- Exact nonce entropy and encoding.
- Whether `expires_at=null` is permitted for long-lived archival messages.
- Event ordering and conflict-resolution rules for concurrent channels.
- Exact signed byte sequence and key lifecycle.
- Whether channel replication uses immutable copies or linked references.
- Retention policy for consumed nonces and expired events.
