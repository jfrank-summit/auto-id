## Auto‑ID Problem Statement: Why Agent Identity Matters for Autonomous, A2A‑Native Systems

This document frames the identity problem for autonomous agents (Auto‑ID), describes why it is pivotal for agent‑to‑agent collaboration, and proposes a research plan toward a pragmatic, interoperable solution.

### Context: Agents are crossing system boundaries

Modern agents are evolving from single‑platform assistants into networked services that coordinate with other agents and systems. Coordination requires more than message delivery: it requires trustworthy identity so peers can authenticate, authorize, and hold each other accountable over long‑running, multi‑modal workflows.

Recent protocols such as Google’s Agent2Agent (A2A) formalize discovery and collaboration patterns. A2A introduces the public, machine‑readable Agent Card (commonly at `/.well-known/agent.json`), standardized authentication declarations, and primitives for tasks, messages, parts, artifacts, streaming, and push notifications. The latest spec highlight support for stateless interactions and OpenAPI‑like authentication schemes, plus JWT‑authenticated push notifications — all of which assume clear, verifiable agent identities.

- Reference: “Understanding A2A — The protocol for agent collaboration” [Google Developer forums](https://discuss.google.dev/t/understanding-a2a-the-protocol-for-agent-collaboration/189103)
- Reference: A2A overview: `https://a2a-protocol.org/latest/topics/what-is-a2a/`
- Reference: A2A specification: `https://a2a-protocol.org/latest/specification/`

### Problem

Without portable, verifiable identity, agents cannot:

- Establish trust across organizational or platform boundaries
- Select compatible authentication schemes at runtime
- Prove authorship of artifacts or updates over long‑running tasks
- Maintain coherent reputation and revocation across redeployments
- Delegate safely (sub‑identities) or rotate keys without breaking trust
- Operate in offline/async modes while preserving verifiable continuity

As multi‑agent ecosystems scale, bespoke identity integrations lead to O(N²) coupling and brittle governance. We need a common, layered identity model that works with A2A’s discovery and auth declarations while remaining flexible about underlying trust roots.

### What “identity” must include for agents

- Stable identifiers: globally unique, dereferenceable where useful
- Authentication: one or more schemes (API key, OAuth2/OIDC, mTLS, JWT/JWK, DID/VC)
- Authorization signals: scopes, capabilities, least‑privilege delegation
- Attestations and reputation: signed claims about behavior, capabilities, or compliance
- Continuity and rotation: safe key rollover, revocation, recovery, compromise handling
- Auditability: signed task updates, artifact integrity, verifiable logs
- Privacy and selective disclosure: minimize linkage, support pseudonymous operation
- Operability: straightforward developer ergonomics, automations, and tooling

### Requirements and constraints

- Interop with A2A Agent Card and auth declarations; discoverability via well‑known endpoints
- Support both synchronous (streaming) and asynchronous (webhook/push) flows
- Multi‑modal, long‑running tasks; offline validation of signatures where possible
- Multiple trust models: centralized, federated, decentralized, and hybrid
- Strong key management: rotation, revocation, hardware support where available
- Enterprise posture: policy, audit, compliance hooks; clear failure and recovery paths
- Performance and cost: lightweight verification, caching, and reasonable latency

### Design space (non‑exclusive)

1. Centralized IDs

   - Examples: API keys, OAuth2/OIDC service accounts
   - Pros: simple, mature tooling; easy enterprise adoption
   - Cons: platform coupling, limited portability across ecosystems

2. Federated/workload identities

   - Examples: cloud workload identities, SPIFFE/SPIRE‑style SVIDs
   - Pros: strong provenance within infra; automatable rotation
   - Cons: federation complexity; cross‑org portability varies

3. Public‑key first (PKI/JWT/JWK)

   - Examples: mTLS, JOSE/JWS with JWKS discovery, ACME‑style automation
   - Pros: offline verification; good fit for signing artifacts and push JWTs
   - Cons: distribution and revocation lists can be operationally heavy

4. Decentralized identity (DID/VC)

   - Examples: DID methods (`did:web`, `did:key`, `did:pkh`), Verifiable Credentials
   - Pros: portability across vendors; selective disclosure; long‑term verifiability
   - Cons: maturity and operational tooling vary; method fragmentation

5. Hybrid
   - Agents advertise multiple schemes in the Agent Card; relying parties select per policy
   - Policy engine governs selection and upgrade paths (e.g., migrate from API key → OIDC → DID)

### Threats and risks (abridged)

- Impersonation and key theft; weak custody on developer machines
- Replay and phishing of webhook endpoints for push notifications
- Supply‑chain compromise of agent images/binaries affecting key trust
- Stale keys and broken rotation leading to silent trust failures
- Over‑privileged scopes and unchecked delegation across agents

### A2A implications for identity

- Agent Card should advertise: identifier(s), service endpoint, supported auth schemes, JWKS (if applicable), capabilities, and skills
- Push notifications should be JWT‑authenticated; `iss`/`sub` should map to stable agent identity; `kid` should resolve via JWKS/DID document
- Artifacts and long‑running task updates benefit from detached signatures for auditability and dispute resolution
- Stateless interactions and streaming favor lightweight, cacheable verification materials
- Prefer storing artifacts and evidentiary materials on the Autonomys DSN via Auto Drive using CIDs; treat this as permanent storage "off" the registry chain

### Minimal viable identity (MVI) for Auto‑ID

1. Identifier

   - Primary: URL or `did:web` for ease of publication, with optional `did:key` for air‑gapped or ephemeral contexts

2. Discovery

   - Publish A2A‑compatible Agent Card at `/.well-known/agent.json`, including supported auth methods and a JWKS URI or DID Document

3. Authentication

   - Baseline adapters: API key, OAuth2 client credentials, mTLS, JWT (with JWKS), and optional DID/VC presentation

4. Signing and audit

   - Sign push notifications (JWT) and optionally task updates/artifacts (detached JWS)
   - Include machine‑verifiable lineage for artifacts; store content on Autonomys DSN (Auto Drive) with CIDs and optional gateway URIs

5. Key lifecycle
   - Automatic rotation, revocation endpoints, incident response playbook, and recovery

### Research plan

Deliver an evidence‑based recommendation and a working reference implementation. Phases:

1. Landscape and criteria (1–2 weeks)

   - Compile evaluation matrix across schemes (security, interop, ops, privacy, cost)
   - Map A2A requirements to identity capabilities; identify gaps

2. Prototypes (2–4 weeks total, can run in parallel)

   - Agent Card generator: library/CLI to emit valid `/.well-known/agent.json` with multi‑scheme auth declarations and optional JWKS or DID Document
   - Auth adapters: pluggable verification for API key, OAuth2/OIDC, mTLS, JWT/JWKS; optional DID/VC verification for `did:web` and `did:key`
   - E2E demo: two agents exchanging messages via A2A `message/send` and `message/stream`, using different identity schemes; JWT‑signed push; artifact signing and verification

3. Usability and ops hardening (1–2 weeks)

   - Key rotation automation; revocation flows; incident drills
   - Caching and offline verification strategy; performance testing

4. Decision and rollout
   - Present trade‑offs and recommended default stack (with fallbacks)
   - Publish reference docs, examples, and checklists for teams

Success criteria

- Interoperates with A2A‑compliant peers using at least two distinct auth schemes
- Supports offline verification of push notifications and artifact signatures
- Demonstrates seamless key rotation without consumer outages
- Minimal developer friction to adopt and operate

### Open questions

- What is the default trust root for first‑party agents vs. third‑party marketplace agents?
- Which DID methods (if any) provide the best portability without heavy infra?
- How should reputation and negative events (abuse, compromise) be modeled and shared?
- What minimal metadata belongs in the Agent Card versus external registries?
- How to model sub‑identities and delegation for tool calls or child workers?

### Next steps

- Approve the research plan and timelines
- Start with the Agent Card generator and JWT/JWKS path (broad interop), then layer OIDC and optional DID/VC
- Define evaluation matrix and target candidates for a2a interop testing

If adopted, Auto‑ID will provide a layered, practical identity foundation: easy to start, capable of centralized and decentralized trust, and aligned with A2A’s discovery and authentication model.

### Related documents

- Blockchain coordination addendum: [design/blockchain-coordination.md](../design/blockchain-coordination.md)
- Agent Card extensions (draft): [design/agent-card-extensions.md](../design/agent-card-extensions.md)
- Identity options evaluation matrix (template): [research/evaluation-matrix.md](../research/evaluation-matrix.md)
