## Auto‑ID: A2A Agent Card Extensions (Draft)

Purpose: propose fields to capture multi‑scheme identity, on‑chain coordinates, and reputation pointers within the A2A Agent Card (e.g., `/.well-known/agent.json`).

### Proposed fields

- `identifiers`: array of identifiers for the agent
  - Examples: `https://agent.example`, `did:web:agent.example`, `did:key:z6Mk...`
- `auth`: declared authentication schemes and parameters
  - Examples: `Bearer` (OAuth2 client credentials), `ApiKey`, `mTLS`, `JWT` (jwks_uri), `DIDVC`
- `jwks_uri`: HTTPS URL for JWKS when using JWT/JWS
- `did_document_uri`: URL to a DID Document (or embedded document)
- `registrations`: ERC‑8004 compatible on‑chain registrations (chain coordinates and proofs)
  - Array entries: `{ chainId, agentId, agentAddress (CAIP‑10), signature }`
- `onChainIdentity`: coordinates to verify rotation and commitments
  - `{ chainId, registry, subjectId, rotationNonce }`
- `reputationCommitment`: locator for the latest attestation Merkle root (contract address + commitment id)
- `reputationEndpoints`: aggregator read APIs for scores and inclusion proofs
- `trustModels`: advertised server‑side trust models (per ERC‑8004): `feedback`, `inference-validation`, `tee-attestation`
- `artifactSigningPolicy`: whether artifacts are signed, and expected algorithms
- `pushJwtRules`: expected claims and issuer/subject mapping for push notifications

### Example (illustrative)

```json
{
  "name": "Example Research Agent",
  "description": "Finds, analyzes, and summarizes documents.",
  "service": { "endpoint": "https://agent.example/a2a" },
  "capabilities": { "streaming": true, "pushNotifications": true },
  "identifiers": ["https://agent.example", "did:web:agent.example"],
  "auth": [
    { "scheme": "ApiKey", "header": "X-API-Key" },
    {
      "scheme": "Bearer",
      "grant": "client_credentials",
      "issuer": "https://auth.example"
    },
    {
      "scheme": "JWT",
      "jwks_uri": "https://agent.example/.well-known/jwks.json"
    }
  ],
  "jwks_uri": "https://agent.example/.well-known/jwks.json",
  "did_document_uri": "https://agent.example/.well-known/did.json",
  "registrations": [
    {
      "chainId": 870,
      "agentId": 12345,
      "agentAddress": "eip155:8453:0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb7",
      "signature": "base64url-jws"
    }
  ],
  "onChainIdentity": {
    "chainId": 8453,
    "registry": "0xRegistryAddress",
    "subjectId": "did:web:agent.example",
    "rotationNonce": 3
  },
  "reputationCommitment": {
    "chainId": 8453,
    "contract": "0xCommitmentContract",
    "commitmentId": "2025-06-01"
  },
  "reputationEndpoints": [
    { "name": "default", "url": "https://reputation.example/api/v1" }
  ],
  "trustModels": ["feedback", "inference-validation", "tee-attestation"],
  "artifactSigningPolicy": { "enabled": true, "alg": ["RS256", "EdDSA"] },
  "pushJwtRules": { "iss": "https://agent.example", "aud": "client-webhook" }
}
```

Notes

- Keep extensions namespaced if the A2A core spec reserves field names (e.g., under `x-auto-id/*`).
- These fields are optional and can be adopted incrementally.
- When these docs refer to "off‑chain" storage, prefer Autonomys DSN via Auto Drive (content‑addressed CIDs) — off the EVM registry chain but permanent and verifiable. Include CIDs and optional `gatewayUri` where helpful.
