## Auto‑ID: Deployment Architectures — EVM v1 vs Autonomys Domain v2

Purpose: outline pragmatic rollout options for Auto‑ID’s on‑chain coordination: a v1 EVM smart‑contract framework and a v2 Autonomys enshrined domain (custom Substrate‑based chain), with migration paths.

### Summary

- v1: EVM contracts (Identity, Reputation, Validation) deployed on an Autonomys Auto EVM domain (EVM runtime). Aligns with ERC‑8004 for maximum interop. Faster to ship, broad tooling.
- v2: Dedicated Autonomys Domain for Auto‑ID, leveraging Decoupled Execution (DecEx) and domain sub‑protocols for native fees, staking, and XDM. Higher control/performance; domain‑native governance; longer lead time.

Reference: Autonomys Domains — see Domains overview [Autonomys Academy](https://academy.autonomys.xyz/autonomys-network/decoupled-execution/domains).

### v1: EVM contracts (Auto EVM)

- Components
  - Identity Registry: maps AgentID→Agent Card URL, CAIP‑10, rotation nonce
  - Reputation Registry: pre‑auth + feedback events; off‑chain evidence with Merkle root commitments
  - Validation Registry: request/response events with `DataHash`
- Pros
  - Rapid delivery; mature infra and wallets
  - ERC‑8004 compatibility for agent ecosystems
  - Easy indexers/analytics
- Cons
  - Gas costs; calldata constraints; limited custom logic for batching/fees
  - DSN references are via external URIs (same for v2 unless special integration is added)

### v2: Autonomys Domain (enshrined L2)

- Domain features
  - Custom runtime pallets for identity, rotation nonce, commitments, and reputation hooks
  - Native fee model and execution fees; staking and slashing via Operator Registry
  - Cross‑Domain Messaging (XDM) to other domains; permissionless operator participation per domain rules
- Pros
  - Flexible data models and batching; fraud proofs and slashing
  - Tailored fee and reputation semantics
- Cons
  - More engineering and operations upfront
  - New explorer/indexer work; validator/operator ecosystem considerations

### Migration and coexistence

- Start on EVM (v1), provide data export and event parity for v2 domain ingest.
- Dual‑write period: EVM emits canonical events; domain mirrors via bridges/indexers until v2 becomes source of truth.
- Agent Card fields support both: `registrations[]` (EVM) and `onChainIdentity` (domain coords).

### API and data compatibility

- Keep event shapes compatible: IdentityRegistered, Rotated, Revoked, AttestationRootCommitted, ValidationRequest/Response
- Hashing and Merkle conventions identical across v1/v2
- URIs always content‑addressed (CIDs) for permanence

### Open questions

- Bridge/indexing path from EVM to domain (light client vs off‑chain indexers)
- Domain fee calibration and incentive alignment for operators
- Governance for domain upgrades and allowlists, if any
