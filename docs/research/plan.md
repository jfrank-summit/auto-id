## Auto‑ID: Detailed Execution Plan (Timeline‑Free)

Purpose: deliver a practical, A2A‑native identity baseline with clear interfaces, working prototypes, and objective acceptance criteria.

### Objectives

- Establish a baseline identity stack aligned with A2A discovery and auth
- Produce reference artifacts (schemas, examples) others can reuse
- Ship minimal, working prototypes for verification, push, and signing
- Keep DID/VC and blockchain coordination optional and additive

### Scope (baseline)

- Identification: HTTPS origin + `/.well-known/agent.json`; optional `did:web`
- Auth: JWT/JWKS (primary), OAuth2 client credentials (enterprise), API key (dev only), mTLS (optional)
- Discovery: A2A Agent Card includes identifiers, supported auth, `jwks_uri`, capabilities
- Signing: push JWTs; optional detached JWS for artifacts
- Key lifecycle: rotation via JWKS `kid`; simple revocation list endpoint
- Reputation: off‑chain signed attestations with evidence URIs; no scoring required

### Workstreams and deliverables

1. Agent Card and discovery

   - Deliverables:
     - Agent Card JSON schema (draft) covering identifiers, auth, `jwks_uri`, capabilities
     - Example `/.well-known/agent.json` aligned with A2A
     - Validation script for schema compliance

2. Auth verification and key lifecycle

   - Deliverables:
     - JWKS publisher (static file or small service) with rotation support
     - Verifier utility: fetch Agent Card → resolve JWKS → verify JWT `iss/sub/kid/alg`
     - Revocations endpoint: `/.well-known/auto-id/revocations.json` and verification logic

3. Push notifications and artifact signing

   - Deliverables:
     - Push sender: emits JWT‑signed callbacks; claims policy documented
     - Push receiver: verifies JWTs against Agent Card/JWKS
     - Artifact signer/verifier: detached JWS over content hash + reference embedding in task updates

4. Reputation attestations (off‑chain)

   - Deliverables:
     - `ReputationAttestation` schema (JSON + JWS profile) and example instances
     - Evidence storage guideline (Autonomys DSN via Auto Drive CIDs + optional gateway URIs + content hash)
     - Consumer policy examples (e.g., minimum issuers, categories, severities)

5. Optional: blockchain coordination (mock first)

   - Deliverables:
     - Mock registry (file or in‑memory) for rotation nonce and commitments
     - Merkle batcher for attestation hashes + inclusion proof verifier
     - Agent Card fields for on‑chain coordinates (enabled but not required)

6. Optional: ERC‑8004 compatibility
   - Deliverables:
     - Agent Card `registrations[]` and `trustModels[]` support
     - Reader/writer utilities for Identity, Reputation, and Validation registries (EVM testnet)
     - Feedback pre‑authorization integration; off‑chain feedback docs with event linkage

### Interfaces and schemas to define

- Agent Card additions: `identifiers[]`, `auth[]`, `jwks_uri`, optional `did_document_uri`, optional on‑chain coords
- JWKS format: preferred algorithms (EdDSA/ES256), `kid` conventions
- Revocations document: array of revoked `kid`s with reasons and timestamps
- ReputationAttestation: issuer, subject, event, task refs, artifacts, evidence, issuedAt/expiresAt
  - Evidence/artifacts should support DSN CIDs and optional `gatewayUri`
- BatchManifest (optional): list of attestation hashes + Merkle metadata

### Prototype layout (suggested)

```
auto-id/
  demo-agent/              # minimal A2A-compatible agent (serves Agent Card + JWKS)
  demo-client/             # calls demo-agent; receives push; verifies JWT
  libs/
    verifier/              # resolve Agent Card, JWKS/DID, verify JWT/JWS
    signing/               # artifact signing/verifying helpers
  schemas/                 # agent card, attestation, revocations, batch manifest
  scripts/                 # validation and rotation helpers
```

### Test plan (acceptance)

- Agent Card validates against schema and passes example validator
- JWT verification succeeds for valid tokens and fails for tampered or stale `kid`
- Key rotation: new `kid` accepted; old `kid` rejected when in revocations
- Push flow: sender → receiver end‑to‑end with JWT verification and claim checks
- Artifact signing: detached JWS verifies against content hash; task update links correctly
- Reputation attestation: JWS verifies; evidence hash matches stored artifact
  (retrieved via Auto Drive by CID)
- Optional Merkle: inclusion proof verifies against committed root (mock)

### Decision gates

- Gate A: Interfaces frozen — Agent Card schema, JWKS/revocations format, attestation schema
- Gate B: Prototypes green — verifier, push sender/receiver, artifact signing, rotation
- Gate C: Recommendation — confirm baseline as default; identify optional add‑ons (OIDC, DID/VC, blockchain)

### Risks and mitigations

- Key distribution pitfalls → cache JWKS with TTL; pin by `kid`; provide revocations doc
- Over‑fitted extensions → keep Agent Card additions namespaced and optional
- Complexity creep → defer DID/VC and blockchain; add only when needed

### Next actions checklist

- [ ] Finalize baseline fields in Agent Card example
- [ ] Write JSON Schema for Agent Card and add validator script
- [ ] Implement JWKS publisher with rotation and revocations doc
- [ ] Build verifier utility (Agent Card → JWKS → JWT verify)
- [ ] Implement push sender/receiver with documented claims
- [ ] Implement artifact signing/verifying and embed into task updates
- [ ] Define ReputationAttestation schema and create examples
- [ ] (Optional) Mock registry + Merkle batcher + inclusion proof verifier
