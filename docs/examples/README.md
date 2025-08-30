## Auto‑ID Examples

### ReputationAttestation

- File: [reputation-attestation.example.json](./reputation-attestation.example.json)
- Shows a signed-object payload structure for a reliability event, referencing artifacts/evidence stored on Autonomys DSN via Auto Drive using CIDs, plus an optional `gatewayUri` for convenience.

Verifier outline (pseudocode)

```text
1. Load attestation JSON
2. Verify JWS signature (if provided) against issuer's keys from Agent Card / JWKS or DID Document
3. Recompute content hash of CID-resolved bytes from Auto Drive; match `hash`
4. Validate required fields (issuer, subject, event, issuedAt)
5. Apply local policy (issuer allowlist, event categories, severity thresholds)
```
