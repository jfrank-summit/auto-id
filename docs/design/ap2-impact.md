## Auto‑ID: Agent Payments Protocol (AP2) — Impact on Design

This note analyzes how Google’s Agent Payments Protocol (AP2) could influence Auto‑ID’s identity, payments, reputation, and validation approach, and how it intersects with A2A. It complements [Problem Statement](../problem/auto-id-problem-statement.md), [Blockchain coordination](./blockchain-coordination.md), and [Payments integration](./payments-integration.md).

### What AP2 proposes (summary)

- Open, payment‑agnostic protocol for agent‑led payments across platforms; can be used as an extension to A2A and MCP
- Addresses three core questions for agentic commerce:
  - Authorization: user’s explicit, scoped permission for an agent to transact
  - Authenticity: merchant confidence that the request reflects the user’s intent
  - Accountability: clear, auditable basis to resolve disputes and fraud
- Introduces cryptographically signed Mandates (verifiable credentials) as evidence:
  - Intent Mandate: user’s instructions and constraints (e.g., budget, timing)
  - Cart Mandate: exact items, price, and terms to be executed
- Supports diverse payment rails (cards, bank transfers, real‑time payments, stablecoins)
- Provides a path for web3 via A2A x402 extension for agent‑based crypto payments
- Open development with a large ecosystem; public specification and references

References:

- AP2 announcement — `https://cloud.google.com/blog/products/ai-machine-learning/announcing-agents-to-payments-ap2-protocol`
- AP2 repository — `http://goo.gle/ap2`
- A2A overview — `https://a2a-protocol.org/latest/topics/what-is-a2a/`
- A2A specification — `https://a2a-protocol.org/latest/specification/`
- A2A x402 (crypto payments) — `https://github.com/google-a2a/a2a-x402`
- Privy X post - `https://x.com/privy_io/status/1978083505100710375`

### Alignment with Auto‑ID baseline

- A2A‑native: AP2 is designed to extend A2A, aligning with Auto‑ID’s Agent Card and discovery at `/.well-known/agent.json`
- Evidence‑first: emphasizes verifiable, tamper‑evident records (Mandates) and an audit trail, consistent with our off‑chain evidence posture
- Chain‑optional: supports traditional rails and crypto; maps to our “blockchain as an additive option” stance
- Interop by design: complements A2A and MCP; fits our goal to remain protocol‑compatible without prescribing a single rail

### Implications for Auto‑ID design

1. Identity, capability discovery, and Agent Card

   - Extend our Agent Card extensions to declare AP2 payments capability and configuration, e.g.:
     - `payments`: `{ protocol: "ap2", mandateTypes: ["intent", "cart"], paymentMethods: ["card", "ach", "rtp", "stablecoin"], x402: true|false, endpoints: { mandatesURI, receiptsURI } }`
     - `complianceProfiles`: e.g., `pci-dss`, `psd2-sca`, `aml-kyc`
   - Keep `onChainIdentity` for non‑EVM and alternative registries; use CAIP‑10 when x402/crypto rails are enabled

2. Authentication, authorization, and keys

   - Baseline unchanged (JWT/JWKS + OIDC) for agent‑to‑agent auth and session security
   - Add support for issuing/verifying Mandates as VCs; bind Mandate subject to the user identity discoverable via Agent Card
   - For delegated tasks, define scoped, expiring Intent Mandates (price caps, categories, time windows); align Mandate key rotation with JWKS `kid` rotation

3. Mandates data model and evidence storage

   - Define canonical JSON schemas for Intent and Cart Mandates and a stable hashing strategy (e.g., JCS + SHA‑256) for evidence integrity
   - Store Mandates and receipts off‑chain; include a `dataHash`/commitment in transaction metadata when needed
   - Maintain a complete audit trail from intent → cart → payment, resolvable via URIs controlled by the agent/operator

4. Payments flows (mapping to Auto‑ID)

   - Real‑time (human present): capture Intent Mandate during user interaction; on approval, sign Cart Mandate; proceed to payment execution
   - Delegated (human not present): pre‑authorize via detailed Intent Mandate; agent generates Cart Mandate when conditions are met
   - Integrate these steps in `payments-integration.md` with sequence diagrams and state transitions; reuse A2A streaming and push JWTs for updates

5. Governance, risk, and operations

   - Respect regional compliance (PCI‑DSS, PSD2/UK SCA, AML/KYC) and PSP requirements; surface `complianceProfiles` in Agent Card
   - Choose rails per use case: traditional PSPs for fiat; x402 for stablecoins/crypto settlements; consider L2s for cost and latency
   - Establish dispute workflows leveraging the audit trail; map outcomes into our `ReputationAttestation` objects where appropriate

6. Privacy

   - Minimize PII/PCI data in Mandates; prefer selective disclosure in VCs when possible
   - Keep detailed payment artifacts off‑chain with access control; publish only hashes/roots as needed

7. Storage with Autonomys DSN / Auto Drive

   - Use DSN (Auto Drive) for `mandatesURI`, `receiptsURI`, and full audit trails; publish content‑addressed CIDs
   - Provide gateway URIs (e.g., `https://gateway.autonomys.xyz/<cid>`) for easy retrieval; ensure `dataHash`/commitments are computed over DSN content bytes

### Recommended stance (incremental adoption)

- Baseline unchanged: keep JWT/JWKS + OIDC, A2A Agent Card, streaming, and JWT‑signed push
- Compatibility layer:
  - Extend Agent Card with `payments` and `complianceProfiles` for AP2
  - Define schemas and hashing for Intent/Cart Mandates; publish resolver endpoints
  - Support AP2 flows end‑to‑end in our payments integration, with optional x402 rails for crypto
- Keep `blockchain-coordination.md` approach (Merkle batching, rotation nonce) as a portable fallback when AP2 is absent

### Updates we should consider in our docs

- `agent-card-extensions.md`: add `payments` block (`protocol`, `mandateTypes`, `paymentMethods`, `x402`, `endpoints`, `complianceProfiles`)
- `payments-integration.md`: document AP2 flows (intent/cart), audit trail, dispute handling, and DSN storage
- `blockchain-coordination.md`: reference AP2/x402 as an alternative to generic registry sketches; clarify hashing/anchoring
- `plan.md`: add optional tasks for AP2 compatibility (Mandate VC support; endpoints; x402 rail integration)
- `glossary.md`: add definitions for AP2, Mandate (Intent, Cart), and x402

### Open questions

- Mandate VC profiles and selective disclosure formats; revocation and expiration semantics
- Identity binding: how Mandate subjects map to A2A identities and OIDC subjects
- Liability and dispute resolution across rails and jurisdictions
- Merchant/PSP adoption timelines and interoperability testing strategy
- Privacy boundaries for storing carts/receipts vs. hashes; user data retention policies

### References

- AP2 announcement — `https://cloud.google.com/blog/products/ai-machine-learning/announcing-agents-to-payments-ap2-protocol`
- AP2 repository — `http://goo.gle/ap2`
- A2A overview — `https://a2a-protocol.org/latest/topics/what-is-a2a/`
- A2A specification — `https://a2a-protocol.org/latest/specification/`
- A2A x402 (crypto payments) — `https://github.com/google-a2a/a2a-x402`
- Intro video — `https://www.youtube.com/watch?v=yLTp3ic2j5c`
