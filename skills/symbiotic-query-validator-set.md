---
name: Query the Symbiotic Relay validator set
description: Read the current epoch and the active validator set from a Symbiotic Relay node, then look up a specific validator.
api: openapi/symbiotic-relay-openapi.json
operations:
  - GetCurrentEpoch
  - GetValidatorSet
  - GetValidatorSetHeader
  - GetValidatorByAddress
  - GetLocalValidator
---

# Query the Symbiotic Relay validator set

The Symbiotic Relay HTTP API is served by each operator's Relay node (default
`http://localhost:8080`, JSON, base64 for binary fields). It exposes no
API-level auth — reach it over the node's trusted network boundary.

## Steps

1. **Get the current epoch** — `GET /v1/epoch/current` (`GetCurrentEpoch`).
   The `epoch` value scopes every validator-set and signature query.
2. **Fetch the validator set** — `GET /v1/validator-set` (`GetValidatorSet`) to
   read the full set (validators, voting power, keys, vaults, quorum threshold).
   Use `GET /v1/validator-set/header` (`GetValidatorSetHeader`) for just the
   header when you only need epoch / capture-timestamp / status.
3. **Look up one validator** — `GET /v1/validator/address/{address}`
   (`GetValidatorByAddress`) to inspect a single operator's voting power, keys,
   and backing vaults.
4. **Identify this node's validator** — `GET /v1/validator/local`
   (`GetLocalValidator`) to confirm the local node's own validator identity.

## Rules

- Treat `epoch` as a string; do not assume it is contiguous — read it from
  `GetCurrentEpoch` rather than incrementing.
- Errors follow the gRPC `Status` envelope (`code`/`message`/`details`), not
  problem+json — see `errors/symbiotic-problem-types.yml`. `NOT_FOUND` (5) means
  the address/epoch is not in the set.
- Binary values (keys, message hashes) are base64 strings.
