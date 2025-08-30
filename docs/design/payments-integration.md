## Auto‑ID: Payments Integration

Purpose: define how agents advertise pricing and accept payments, and how clients prove payment for [A2A](../glossary.md#a2a-agent-to-agent) interactions and long‑running tasks, with support for both fiat processors and on‑chain payments.

### Goals

- Portable, verifiable payment signals across orgs and platforms
- Minimal friction for common cases (usage‑based, per‑task, subscription)
- Clear linkage between payment, identity, and task/artifact lineage
- Optional on‑chain settlement without forcing blockchain usage

### Design overview

- Agents publish a Payment Policy in their Agent Card, including supported methods, currencies/tokens, pricing models, and billing endpoints.
- Charges are tied to A2A task identifiers and artifact IDs; invoices and receipts are signed ([JWS](../glossary.md#jws-json-web-signature)) and can be validated offline.
- Fiat and crypto payment adapters are pluggable; receipts normalize to a common schema.
- For crypto, small payments can use streaming or pre‑funded allowances; larger ones use invoice/escrow patterns.

### Agent Card additions

- `payments`: high‑level policy and supported methods
  - `{ methods: ["fiat:stripe", "crypto:eip155:8453:USDC", "crypto:btc:lightning"], currencies: ["USD", "USDC"], pricing: { model: "usage|per_task|subscription", unit: "token|request|artifact|minute", rate: 0.002, currency: "USD" } }`
- `billingEndpoints`: URLs for billing APIs
  - `{ invoices: "https://agent.example/billing/invoices", receipts: "https://agent.example/billing/receipts", usage: "https://agent.example/billing/usage" }`
- `accepts`: optional allowlist/denylist for jurisdictions or counterparties
- `escrow`: optional coordinates for on‑chain escrow contracts (when used)

See `design/agent-card-extensions.md` for the full example.

### Runtime flows

1. Usage‑based (post‑paid)

- Client starts task; agent meters usage; agent issues signed Invoice JWS referencing `task.id` and metering summary. Client pays within terms; agent issues signed Receipt JWS.

2. Pre‑pay credit (account or allowance)

- Client funds balance via fiat or crypto. Agent issues a signed Credit Receipt. Requests include a bearer token bound to the funded account; usage is deducted. Low‑balance push notifications use signed JWT.

3. Per‑task fixed price

- Agent quotes price for scope; client approves and pays; agent issues Receipt and begins execution. Refund logic is policy‑driven.

4. Crypto streaming/pay‑per‑second (optional)

- For long‑running streaming tasks, use streaming payments (e.g., Superfluid) or short‑interval metered invoices. The Agent Card declares support and minimum interval.

5. Aggregated micropayments

- For high‑frequency, low‑value interactions, use off‑chain signed receipts that periodically aggregate into a single on‑chain redeemable voucher. The receiver (or an aggregator) accepts signed receipts per request and issues an aggregated voucher that is redeemed on‑chain under risk bounds configured by the receiver. This reduces on‑chain costs while maintaining cryptographic accountability.

### Data structures (JWS payloads)

- Invoice
  - `{ type: "Invoice", version: "v1", issuer, subject, task: { id }, lineItems: [...], subtotal, currency, dueAt, createdAt, paymentMethods: [...], references: { agentCard, pricing }, id }`
- Receipt
  - `{ type: "Receipt", version: "v1", issuer, subject, task: { id }, invoiceId, amount, currency, method, txRef, createdAt, id }`
- Credit Receipt

  - `{ type: "CreditReceipt", version: "v1", issuer, subject, amount, currency, method, txRef, createdAt, balanceAfter, id }

- Micropayment Receipt (off‑chain)
  - `{ type: "MicroReceipt", version: "v1", sender, receiver, task: { id }, unit, amount, currency, createdAt, nonce, sessionId, aggregator?, id }`
- Aggregated Voucher (redeemable)
  - `{ type: "AggregateVoucher", version: "v1", receiver, receipts: { count, amount }, currency, window: { start, end }, sessionId, redeemChain, contract, redeemDataHash, createdAt, last, id }`

All JWS `issuer` keys resolve via Agent Card `jwks_uri` or DID Document; `subject` is the paying agent/client identifier.

### APIs (sketch)

- `POST /billing/invoices` → create invoice for task; returns Invoice JWS
- `GET /billing/invoices/{id}` → fetch invoice
- `POST /billing/receipts` → submit payment proof (processor webhooks or tx refs); returns Receipt JWS
- `GET /billing/receipts/{id}` → fetch receipt
- `GET /billing/usage?taskId=*` → metering summary (signed JSON)
- `POST /billing/micro/receipt` → submit signed micro‑receipt (off‑chain)
- `POST /billing/micro/aggregate` → request/update aggregated voucher for a session

### Crypto specifics (optional)

- Supported tokens advertised as CAIP‑19 (`eip155:8453/slip44:60` for ETH derivatives) and stablecoins (e.g., USDC on Base: `eip155:8453/erc20:0x...`).
- Wallets/escrow as CAIP‑10 addresses; proofs of control provided via EIP‑191 message signatures.
- For escrow, contract ABI and chain ID are included in `escrow` with link to policy.
- Aggregated vouchers redeemed on configured chain; risk parameters bound by policy.

### Security and compliance

- Signed invoices/receipts; strict binding to `task.id` and Agent Card identifiers
- Webhooks for processors (e.g., Stripe) must be JWT‑authenticated and replay‑protected
- Privacy: invoices/receipts omit sensitive PII; store minimal details off‑chain; evidence links via CIDs when needed
- Micropayments: enforce nonce/session anti‑replay; cap receiver exposure via configured max outstanding amount; aggregate windows signed by receiver

### Interop with reputation

- Optionally emit `ReputationAttestation` events for payment reliability (e.g., timely payment) or disputes, referencing invoice/receipt IDs and task IDs.

### Developer experience

- Libraries
  - Invoice/receipt JWS builders and verifiers
  - Adapters: Stripe, Coinbase Commerce, EVM ERC‑20 transfer checker, Lightning (optional)
- CLI
  - `auto-id billing invoice --task <id> --amount <n> --currency USD`
  - `auto-id billing verify-receipt <file>`

### Open questions

- Refund and chargeback policies across methods
- Micropayment thresholds and fallback to batching
- Tax and jurisdictional considerations for marketplaces
