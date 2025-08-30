## Auto‑ID: Identifiers and Uniqueness

Purpose: define how Auto‑ID ensures globally unique, verifiable identifiers across discovery, auth, tasks, artifacts, and sub‑identities.

### Identifier namespaces

- HTTPS origin (primary)

  - Canonical identifier: the agent's HTTPS origin (e.g., `https://agent.example`) that hosts `/.well-known/agent.json`.
  - Uniqueness is delegated to DNS/CA; control is proven by serving the Agent Card and keys under that origin.

- DID methods (optional)

  - `did:web:<domain>` maps 1:1 with HTTPS origin; canonicalizes to lowercase host; proves control via hosted DID Document.
  - `did:key:<multibase>` is self‑certifying and unique by key fingerprint; best for ephemeral or air‑gapped contexts.

- On‑chain coordinates (optional)

  - Accounts and assets use CAIP‑10/CAIP‑19 (e.g., `eip155:8453:0x...`, `eip155:8453/erc20:0x...`).
  - Optional ERC‑8004 registries provide handles and rotation nonces; uniqueness is enforced by chain consensus.

- Content identifiers
  - Artifacts/evidence use CIDs (content‑addressed hashes). CIDs are collision‑resistant and therefore globally unique for given content bytes.

### Recommended identifier set in Agent Card

- `identifiers[]`: include both `https://<origin>` and `did:web:<origin>`; optionally add `did:key:` for offline use.
- `service.endpoint`: HTTPS URL for A2A endpoint under the same origin.
- `jwks_uri`: HTTPS URL under the same origin; keys identified by unique `kid`s.

### Sub‑identities and delegation

- Use URI fragments or path suffixes bound to the primary identifier: `https://agent.example#tool:browser`, `https://agent.example/sub/worker-123`.
- Advertise delegated keys/policies in the Agent Card (future extension) or via capability tokens referencing the fragment.

### Key identifiers (kid) and rotation

- `kid` must be unique within the JWKS and stable for the lifetime of the key.
- Suggested format: `<alg>-<createdIso>-<fingerprint>` where `fingerprint = base64url(sha256(spki))`.
- Rotation is communicated via JWKS updates and (optionally) a rotation nonce in on‑chain coordinates; revoked `kid`s appear in `/.well-known/auto-id/revocations.json`.

### Task and request identifiers

- Use time‑ordered, globally unique IDs: UUIDv7 or ULID.
- Bind all invoices/receipts, attestations, and artifacts to `task.id` to prevent replay and mix‑up.

### Canonicalization rules

- HTTPS identifiers: lowercase host; remove default ports; preserve path; strip trailing slash except origin.
- DID identifiers: follow method canonicalization rules; lowercase host for `did:web`.
- CAIP identifiers: preserve case for hex addresses exactly as provided; include explicit chain ID.

### Validation and enforcement

- Authoring time: linter/validator checks for
  - Presence of `identifiers[]` with an origin URL and `did:web` equivalent
  - Reachable `jwks_uri` under the same origin
  - Unique `kid` values within JWKS; revocations well‑formed
  - Canonicalization (no mixed‑case hosts, no trailing slashes on origins)
- Runtime: verifiers enforce
  - `iss`/`sub` must match one of `identifiers[]`
  - `kid` must resolve from `jwks_uri` and not be revoked
  - Optional rotation nonce freshness when on‑chain coordinates are present

### Examples

```json
{
  "identifiers": ["https://agent.example", "did:web:agent.example"],
  "service": { "endpoint": "https://agent.example/a2a" },
  "jwks_uri": "https://agent.example/.well-known/jwks.json"
}
```

Task ID example: `018f4f37-2b31-7c6f-b6a0-9a5b7a72c9f1` (UUIDv7)

Sub‑identity example: `https://agent.example#tool:retriever`

### Rationale

Combining authority‑based (HTTPS/DNS), self‑certifying (DID/key), consensus‑based (CAIP/on‑chain), and content‑addressed (CID) identifiers provides layered uniqueness with verifiable control and smooth offline/online operation.
