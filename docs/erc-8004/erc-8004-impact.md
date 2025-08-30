## Auto‑ID: ERC‑8004 “Trustless Agents” — Impact on Design

This note analyzes how ERC‑8004 (Trustless Agents) could influence Auto‑ID’s identity, reputation, and validation approach, and how it intersects with [A2A](../glossary.md#a2a-agent-to-agent). It complements [Problem Statement](../problem/auto-id-problem-statement.md) and [Blockchain coordination](../design/blockchain-coordination.md).

### What ERC‑8004 proposes (summary)

- Three lightweight, on‑chain registries that extend A2A for open, cross‑org trust:
  - Identity Registry: minimal on‑chain handle resolving to Agent Card, with `AgentID`, `AgentDomain` (RFC 8615 well‑known path), and `AgentAddress` (CAIP‑10)
  - Reputation Registry: pre‑authorization + feedback flow; store minimal on‑chain data, keep rich feedback off‑chain
  - Validation Registry: generic hooks for inference validation (crypto‑economic) and cryptographic verification (e.g., [TEEs](../glossary.md#tee-trusted-execution-environment), [zk](../glossary.md#zk-zero-knowledge))
- Agents advertise in the Agent Card a `registrations` array (on‑chain entries) and supported `trustModels` for server roles

References:

- ERC‑8004: Trustless Agents — `https://eips.ethereum.org/EIPS/eip-8004`
- Discussion: `https://ethereum-magicians.org/t/erc-8004-trustless-agents/25098`
- A2A overview: `https://a2a-protocol.org/latest/topics/what-is-a2a/`
- A2A spec: `https://a2a-protocol.org/latest/specification/`
- A2A forum article: `https://discuss.google.dev/t/understanding-a2a-the-protocol-for-agent-collaboration/189103`

### Alignment with Auto‑ID baseline

- A2A‑native: ERC‑8004 explicitly extends A2A, aligning with Auto‑ID’s baseline (Agent Card at `/.well-known/agent.json`, streaming, push JWTs, tasks/artifacts)
- Evidence‑first: both approaches keep rich evidence off‑chain and put minimal commitments or pointers on‑chain
- Optional blockchain: matches Auto‑ID’s stance to keep chain coordination additive, not mandatory

### Implications for Auto‑ID design

1. Identity and discovery

   - Add ERC‑8004’s `registrations` array to our Agent Card extensions, including CAIP‑10 addresses and ownership proofs
   - Keep our existing `onChainIdentity` pointer for non‑EVM registries or alternative designs; map between the two
   - Consider exposing `trustModels` in Agent Cards for selection (e.g., `feedback`, `inference-validation`, `tee-attestation`)

2. Authentication and keys

   - No change to baseline (JWT/JWKS + OIDC). ERC‑8004 does not mandate auth changes; keys remain discoverable via Agent Card
   - If using on‑chain rotation semantics, reflect updates through registry events and synchronize with JWKS `kid` rotation

3. Reputation

   - Our current plan: signed, off‑chain `ReputationAttestation` objects with optional Merkle commitments on‑chain
   - ERC‑8004’s Reputation Registry introduces pre‑authorization (`AcceptFeedback`) and minimal on‑chain events
   - Interop path: when present, honor pre‑authorization before accepting client feedback; store feedback off‑chain and optionally anchor via Merkle root (ours) or emit compatible events (ERC‑8004)

4. Validation

   - Our plan: optional Merkle commitments for attestations; no mandated validation protocol
   - ERC‑8004 adds generic `ValidationRequest/Response` with `DataHash` committing to re‑execution or TEE/zk proofs
   - Interop path: optionally expose `ValidationRequestsURI`/`ValidationResponsesURI` in Agent Cards; support reading/writing ERC‑8004 validation events while keeping evidence off‑chain

5. Governance and operations

   - ERC‑8004 is chain‑agnostic (L2 or mainnet) and uses CAIP‑10; fits Auto‑ID’s chain‑optional posture
   - For production, we can target a low‑cost L2 for registry writes; checkpointing to L1 is optional and use‑case driven

6. Privacy

   - Both designs prefer off‑chain storage for detailed feedback/validation data; publish hashes/roots on‑chain if needed
   - DID/VC selective disclosure remains compatible for privacy‑preserving signals

7. Storage with Autonomys DSN / Auto Drive

   - Where ERC‑8004 references off‑chain URIs (e.g., `FeedbackDataURI`, `ValidationRequestsURI`, `ValidationResponsesURI`), use Autonomys DSN via Auto Drive with content‑addressed CIDs
   - Publish gateway URIs (e.g., `https://gateway.autonomys.xyz/<cid>`) alongside CIDs for easy retrieval; ensure `DataHash`/hash commitments are computed over the DSN content bytes
   - This preserves permanence and auditability while keeping on‑chain writes minimal

### Recommended stance (incremental adoption)

- Baseline unchanged: ship JWT/JWKS + OIDC, A2A Agent Card with `jwks_uri`, streaming, and JWT‑signed push
- Compatibility layer:
  - Extend Agent Card per ERC‑8004 with `registrations` and `trustModels`
  - Add resolvers to read Identity Registry (when configured) and cross‑check `AgentDomain` → Agent Card
  - Optionally emit/consume Reputation Registry pre‑authorization and feedback events; keep feedback content off‑chain
  - Optionally emit/consume Validation Registry events; keep `DataHash` resolvable via URIs you control
- Keep our `blockchain-coordination.md` approach (Merkle batching, rotation nonce) as a portable fallback when ERC‑8004 is absent

### Updates we should consider in our docs

- `agent-card-extensions.md`: add `registrations[]` and `trustModels[]` fields; note CAIP‑10 and proofs of address ownership
- `blockchain-coordination.md`: mention ERC‑8004 registries as an alternative to our generic registry sketches; clarify mapping
- `plan.md`: add optional tasks for ERC‑8004 compatibility (read/write to registries; agent card field support)

### Open questions

- Chain selection and fees; who operates registries in target ecosystems
- Verification of `AgentDomain` without an oracle (ERC‑8004 leaves this to users)
- Coexistence with DID methods and when to favor DID over on‑chain registry handles
- Security model for pre‑authorization and potential abuse of feedback flows

### References

- ERC‑8004: Trustless Agents — `https://eips.ethereum.org/EIPS/eip-8004`
- Discussion thread — `https://ethereum-magicians.org/t/erc-8004-trustless-agents/25098`
- A2A overview — `https://a2a-protocol.org/latest/topics/what-is-a2a/`
- A2A specification — `https://a2a-protocol.org/latest/specification/`
- A2A community article — `https://discuss.google.dev/t/understanding-a2a-the-protocol-for-agent-collaboration/189103`
