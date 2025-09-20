## Auto‑ID: Blockchain Coordination Addendum

Purpose: define how a blockchain can coordinate agent identity, reputation, and negative‑event sharing in a way that complements [A2A](../glossary.md#a2a-agent-to-agent).

### Goals

- Interoperate with A2A: discovery via Agent Card, standard auth schemes, [JWT](../glossary.md#jwt-json-web-token)‑signed push, long‑running tasks
- Provide portable, verifiable signals: identity anchoring, revocation, attestations, and aggregated reputation
- Keep costs low by anchoring commitments on‑chain while storing rich evidence off‑chain (or permanent storage such as Autonomys DSN)
- Interoperate with ERC‑8004 registries when present; provide mappings and fallbacks

### On‑chain registry interface (minimal)

Note: This generic interface maps to ERC‑8004 Identity and Reputation registries; see the ERC‑8004 compatibility section below.

- Subject identifier: URL or [DID](../glossary.md#did-decentralized-identifier) (e.g., `did:web`), optional additional ids (`did:key`, `did:pkh`)
- Verification material pointer: [JWKS](../glossary.md#jwks-json-web-key-set) URI or DID Document hash
- Rotation nonce: monotonically increasing integer to invalidate prior keys
- Optional stake and policy flags: for governance or risk weighting

Key events

- Registered(subjectId, jwksUri|didDocHash)
- Rotated(subjectId, newJwksUri|didDocHash, newNonce)
- Revoked(subjectId, reasonCode, evidenceCommitment)
- AttestationRootCommitted(commitmentId, merkleRoot, period)

### Attestations and reputation

Evidence‑first model: counterparties issue signed `ReputationAttestation` objects (JWS) that reference task ids, artifacts, and evidence URIs (off‑chain, permanent storage). Only the content hash (and optional small metadata) is anchored on‑chain, typically via periodic Merkle root commitments.

Reputation is computed off‑chain by one or more aggregators that publish:

- Aggregation algorithm and parameters
- Current score snapshot with inclusion proofs against the latest Merkle root

Suggested attestation fields (JSON)

```json
{
  "type": "ReputationAttestation",
  "schema": "https://auto.id/schemas/reputation-attestation/v1",
  "issuer": "did:web:issuer.example",
  "subject": "did:web:agent.example",
  "event": { "category": "safety", "name": "policy_violation", "severity": 3 },
  "task": { "id": "a2a-task-uuid", "contextId": "optional-context" },
  "artifacts": [
    { "uri": "https://permanent.storage/cid", "hash": "sha256:..." }
  ],
  "evidence": [
    { "uri": "https://permanent.storage/cid2", "hash": "sha256:..." }
  ],
  "issuedAt": "2025-06-01T12:00:00Z",
  "expiresAt": null
}
```

### Merkle commitment process

- Collect attestations over a period (e.g., hourly)
- Build a Merkle tree over attestation content hashes
- Commit `merkleRoot` to the registry; emit `AttestationRootCommitted`
- Publish the batch manifest and per‑attestation Merkle proofs

### Revocation and rotation

- Rotation: publish new `kid` via JWKS or DID Doc; increment on‑chain rotation nonce
- Revocation: emit on‑chain record referencing evidence commitment; verifiers reject JWTs signed with keys older than the current nonce

### Agent Card additions (see companion doc for full schema)

- `identifiers`: array of URL/DID forms
- `auth`: declared schemes (API key, OAuth2, mTLS, JWT/JWKS, DID/VC)
- `jwks_uri` or `did_document_uri`
- `onChainIdentity`: `{ chainId, registry, subjectId, rotationNonce }`
- `reputationCommitment`: latest commitment locator
- `reputationEndpoints`: read APIs for aggregators and proof retrieval

### Verifier rules (sketch)

1. Resolve Agent Card, fetch `jwks_uri` or DID Document
2. Fetch on‑chain rotation nonce; reject signatures with older nonce
3. Validate push JWTs and artifact signatures; require `iss`/`sub` to match advertised identifiers
4. When consuming reputation: verify inclusion proof against the latest committed Merkle root; apply local policy

### Privacy and selective disclosure

- Use VCs with selective disclosure to prove “score ≥ threshold” without leaking all attestations
- Keep raw evidence controlled; anchor only hashes and batch roots on‑chain

### Chain selection and cost

- Prefer L2/appchain for frequent commits; periodically checkpoint to L1
- Batch commitments; leverage calldata‑efficient encodings

### ERC‑8004 compatibility (optional)

- Identity: when an ERC‑8004 Identity Registry is available, add `registrations[]` in Agent Card (CAIP‑10 address + signature) and optionally map to `onChainIdentity`
- Reputation: honor pre‑authorization events and emit feedback events if using the ERC‑8004 Reputation Registry; keep feedback content off‑chain and anchor via Merkle roots if desired
- Validation: support publishing `ValidationRequestsURI`/`ValidationResponsesURI` and interacting with `ValidationRequest/Response` events while keeping `DataHash` resolvable via permanent URIs
- Discovery: include `trustModels[]` to communicate which trust paths are supported by the server agent

See also: [ERC‑8004 impact analysis](../erc-8004/erc-8004-impact.md) and [AP2 impact analysis](./ap2-impact.md)

### Open questions

- What governance model maintains the registry (DAO, multi‑sig, enterprise CA)?
- Which reputation dimensions are in‑scope (safety, reliability, compliance)?
- How are disputes and appeals recorded and resolved?

### References

- Google Developer forums: “Understanding A2A — The protocol for agent collaboration” (https://discuss.google.dev/t/understanding-a2a-the-protocol-for-agent-collaboration/189103)
- A2A overview: https://a2a-protocol.org/latest/topics/what-is-a2a/
- A2A specification: https://a2a-protocol.org/latest/specification/
- ERC‑8004: Trustless Agents — https://eips.ethereum.org/EIPS/eip-8004
- Discussion: https://ethereum-magicians.org/t/erc-8004-trustless-agents/25098
- ERC‑8004 impact analysis: ../erc-8004/erc-8004-impact.md
