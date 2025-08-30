## Auto‑ID Documentation

### Overview

**Auto‑ID** is an identity and reputation system for autonomous AI agents that enables secure, verifiable agent‑to‑agent collaboration across organizational and platform boundaries.

#### What Auto‑ID provides

- **Portable Identity**: Agents get globally unique, verifiable identities that work across platforms and organizations
- **Multi‑Scheme Authentication**: Support for API keys, OAuth2/OIDC, mTLS, JWT/JWKS, and optional DID/VC authentication
- **A2A‑Native**: First‑class integration with Google's Agent‑to‑Agent protocol for discovery, streaming, push notifications, and task coordination
- **Reputation & Attestations**: Off‑chain reputation system with cryptographic attestations and optional blockchain anchoring
- **Key Lifecycle Management**: Automated key rotation, revocation, and recovery with enterprise‑grade security
- **Permanent Storage**: Integration with Autonomys DSN for immutable artifact and evidence storage

#### Core components

- **Agent Card Extensions**: Enhanced `/.well-known/agent.json` with identity, auth schemes, and reputation pointers
- **Verification Libraries**: Pluggable auth adapters for different identity schemes and JWT/artifact verification
- **Reputation System**: Signed attestations with Merkle tree commitments and aggregation APIs
- **CLI & Tooling**: Developer tools for identity generation, key management, and A2A integration
- **Reference Implementations**: Demo agents and integration examples

#### Use cases

- **Cross‑Platform Agent Collaboration**: Agents from different vendors securely working together on complex tasks
- **Enterprise Agent Governance**: Policy‑driven identity and access management for internal agent fleets
- **Agent Marketplaces**: Verifiable reputation and identity for agents offering services to other agents
- **Autonomous Organizations**: AI agents participating in DAOs with verifiable voting and execution rights
- **Long‑Running Workflows**: Multi‑step processes with cryptographic audit trails and artifact integrity

### Document map

- Problem framing: [problem/auto-id-problem-statement.md](./problem/auto-id-problem-statement.md)
- Agent Card extensions (draft): [design/agent-card-extensions.md](./design/agent-card-extensions.md)
- Identifiers & uniqueness: [design/identifiers-and-uniqueness.md](./design/identifiers-and-uniqueness.md)
- Blockchain coordination addendum: [design/blockchain-coordination.md](./design/blockchain-coordination.md)
- ERC‑8004 impact analysis: [erc-8004/erc-8004-impact.md](./erc-8004/erc-8004-impact.md)
- Payments integration: [design/payments-integration.md](./design/payments-integration.md)
- Identity options evaluation matrix (template): [research/evaluation-matrix.md](./research/evaluation-matrix.md)
- Potential execution plan: [research/plan.md](./research/plan.md)

### Examples

- See [examples/](./examples/) and [examples/README.md](./examples/README.md)

### Schemas

- Reputation Attestation JSON Schema: [schemas/reputation-attestation.schema.json](./schemas/reputation-attestation.schema.json)

### References

- Background: [references/agent-autonomy.md](./references/agent-autonomy.md)
- Centralization limits: [references/centralization-limits-agent-autonomy.md](./references/centralization-limits-agent-autonomy.md)
- Glossary: [glossary.md](./glossary.md)

### Suggested reading order

1. Problem statement
2. Agent Card extensions
3. Blockchain coordination
4. Evaluation matrix
5. Plan
