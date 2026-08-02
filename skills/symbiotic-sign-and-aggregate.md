---
name: Sign a message and collect its aggregation proof
description: Submit a message to the Symbiotic Relay for signing, then poll aggregation status and retrieve the aggregated proof.
api: openapi/symbiotic-relay-openapi.json
operations:
  - SignMessage
  - GetSignatureRequest
  - GetAggregationStatus
  - GetSignatures
  - GetAggregationProof
---

# Sign a message and collect its aggregation proof

Use the Relay node's HTTP API to request a validator-set signature over a message
and obtain the aggregated proof that downstream contracts verify.

## Steps

1. **Submit the message** — `POST /v1/sign` (`SignMessage`) with the message
   (base64) and key tag. The response returns a `requestId`.
2. **Confirm the request** — `GET /v1/signature-request/{requestId}`
   (`GetSignatureRequest`) to read back the request, its `keyTag`, and
   `requiredEpoch`.
3. **Poll aggregation status** — `GET /v1/aggregation/status/{requestId}`
   (`GetAggregationStatus`) until aggregation completes (quorum reached).
   Optionally inspect individual operator signatures via
   `GET /v1/signatures/{requestId}` (`GetSignatures`).
4. **Retrieve the proof** — `GET /v1/aggregation/proof/{requestId}`
   (`GetAggregationProof`) to fetch the aggregated `proof` and `messageHash`.

## Rules

- `POST /v1/sign` is the only write operation and is **not** idempotent — there
  is no Idempotency-Key contract (`conventions/symbiotic-conventions.yml`); track
  the returned `requestId` yourself and reuse it rather than re-submitting.
- Poll `GetAggregationStatus` with backoff; `FAILED_PRECONDITION` (9) means the
  required epoch is not yet committed. For push instead of poll, subscribe to
  `GET /v1/stream/proofs` (`ListenProofs`).
- All message/signature/proof fields are base64-encoded strings.
