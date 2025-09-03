## Auto‑ID Milestones

High‑level milestones to move from design to a usable reference implementation. Sequence matters more than exact timing.

### M0 — Repo & Docs

- Repo structure, docs baseline, glossary
- Problem statement, Agent Card extensions, identifiers/uniqueness
- Plan and milestones published

### M1 — Agent Card & Verifier

- Agent Card JSON Schema + example generator
- JWKS publisher with rotation + revocations doc
- Verifier library: resolve Agent Card → JWKS/DID → verify JWT

Goals: Example agent serves Agent Card + JWKS; verifier validates JWT end‑to‑end.

### M2 — Push & Artifact Signing

- Push sender/receiver with claims policy
- Detached JWS signer/verifier for artifacts; embed refs in task updates

Goals: Demo shows signed push and artifact verification.

### M3 — Reputation Attestations

- ReputationAttestation schema + examples
- Merkle batcher (mock) + inclusion proof verifier

Goals: Attestations validated; inclusion proofs verify against committed root (mock).

### M4 — Payments

- Billing endpoints (invoices, receipts, usage)
- Adapters: Stripe (fiat), ERC‑20 transfer checker (USDC), optional aggregated micropayments

Goals: End‑to‑end task with invoice → payment → receipt.

### M5 — Optional: ERC‑8004 Compatibility

- `registrations[]`, `trustModels[]` in Agent Card
- Reader/writer utilities for registries on Auto EVM

Goals: Demo reads/writes registry events on testnet.

### M6 — Optional: On‑chain Coordination

- Mock registry → EVM v1 contract deployment (Auto EVM)
- Rotation nonce + attestation root commitments

Goals: Registry events consumed by verifier and demo.

### M7 — Deployment Strategy

- v1 EVM deployment plan (Auto EVM); v2 domain draft
- Dual‑write/migration plan and cutover criteria

Goals: Go/No‑Go on v1 deploy; v2 scope agreed.
