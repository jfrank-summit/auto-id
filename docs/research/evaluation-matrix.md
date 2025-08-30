## Auto‑ID: Identity Options Evaluation Matrix (Template)

Scoring guidance: 1 (weak) → 5 (strong). Add qualitative notes where nuance matters.

| Scheme                        | Security | Interop with A2A | Key lifecycle (rotation/revocation) | Privacy/selective disclosure | Operational complexity | Cost/latency | Ecosystem maturity | Tooling/DevEx | Pros                                    | Cons                                  | Notes |
| ----------------------------- | -------- | ---------------- | ----------------------------------- | ---------------------------- | ---------------------- | ------------ | ------------------ | ------------- | --------------------------------------- | ------------------------------------- | ----- |
| API Key                       | 2        | 4                | 2                                   | 1                            | 2                      | 5            | 5                  | 5             | Simple, ubiquitous                      | Weak revocation/rotation; coarse auth |       |
| OAuth2/OIDC                   | 4        | 5                | 4                                   | 2                            | 3                      | 4            | 5                  | 4             | Enterprise‑ready; well‑understood       | Infra coupling; token plumbing        |       |
| PKI/JWT/JWKS                  | 4        | 5                | 4                                   | 2                            | 3                      | 4            | 5                  | 4             | Offline verification; great for signing | JWKS distribution/rotation ops        |       |
| DID/web                       | 4        | 4                | 4                                   | 3                            | 3                      | 4            | 4                  | 3             | Easy publication; portable              | Ties to domain control                |       |
| DID/key                       | 4        | 3                | 3                                   | 3                            | 3                      | 5            | 3                  | 3             | No infra dependency                     | Discovery and rotation UX             |       |
| DID/VC (selective disclosure) | 5        | 3                | 4                                   | 5                            | 4                      | 3            | 3                  | 3             | Strong privacy & claims                 | More complex issuance/verify          |       |

Decision notes

- Use multiple schemes in parallel; select per relying‑party policy
- Start with JWT/JWKS + OIDC; layer DID/VC where needed
- Prefer lightweight verification for streaming and push flows
