# Auto‑ID

**Identity and reputation system for autonomous AI agents**

Auto‑ID enables secure, verifiable agent‑to‑agent collaboration across organizational and platform boundaries through portable identity, multi‑scheme authentication, and cryptographic reputation.

## What Auto‑ID provides

- **Portable Identity**: Globally unique, verifiable identities that work across platforms
- **A2A‑Native**: First‑class integration with Google's Agent‑to‑Agent protocol
- **Multi‑Scheme Auth**: API keys, OAuth2/OIDC, mTLS, JWT/JWKS, and optional DID/VC
- **Reputation System**: Off‑chain attestations with optional blockchain anchoring
- **Payment Integration**: Fiat and crypto billing and payments
- **Key Lifecycle**: Automated rotation, revocation, and recovery

## Documentation

See [`docs/`](./docs/) for complete documentation, including:

- [Product Overview](./docs/README.md)
- [Problem Statement](./docs/problem/auto-id-problem-statement.md)
- [Agent Card Extensions](./docs/design/agent-card-extensions.md)
- [Payments Integration](./docs/design/payments-integration.md)
- [AP2 impact analysis](./docs/design/ap2-impact.md)
- [Deployment Architectures](./docs/design/deployment-architectures.md)
- [Glossary](./docs/glossary.md)

## Repository Structure

```
auto-id/
  docs/                    # Design docs and specifications
    problem/               # Problem framing
    design/                # Technical design docs
    research/              # Evaluation and planning
    examples/              # Usage examples
    schemas/               # JSON schemas
    references/            # Background material
  packages/                # TypeScript libraries (planned)
  apps/                    # Demo applications (planned)
  examples/                # Integration examples (planned)
```

## Quick Start

1. **Read the docs**: Start with the [Problem Statement](./docs/problem/auto-id-problem-statement.md)
2. **Explore design**: Review [Agent Card Extensions](./docs/design/agent-card-extensions.md)
3. **Check the plan**: See [Execution Plan](./docs/research/plan.md)

## Status

🚧 **In Development** — Currently in design and planning phase. Implementation coming soon.

## Contributing

This project is part of the Autonomys ecosystem. Feedback and contributions welcome.
