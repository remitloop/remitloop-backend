# RemitLoop — remitloop-backend

> Cross-border payment state tracking and reconciliation.

![RemitLoop Backend API](banner.png)

## About the project

RemitLoop tracks cross-border payments (remittances) from start to finish on Stellar/Soroban. Each transfer has a state that moves through its lifecycle, and the sending and receiving sides can reconcile their records against one shared, tamper-evident source instead of trading spreadsheets and emails. The loop closes when both sides agree a payment has settled.

**Who it is for:** Remittance operators, payment partners on both sides of a corridor, and the senders and recipients who want to know where their money is.

**How the pieces fit together:**

| Repository | Responsibility |
|---|---|
| `remitloop-contracts` | On-chain Soroban state and authorization — the source of truth |
| `remitloop-backend` | Off-chain indexing, read models and operational APIs |
| `remitloop-app` | User-facing web application |

Typical flow:

1. A payment's state is recorded and advanced on-chain by an authorized party (authorized by that party's Stellar account).
2. The backend indexes state changes into a read model for tracking and reconciliation reports.
3. Operators and customers open the web app to follow a payment and see whether both sides reconcile.

## This repository: Backend API

The **backend** repository is the off-chain service for RemitLoop. It indexes contract activity, serves read models and exposes operational APIs so the web app stays fast. By design it is *not* the source of truth for anything the contract owns — if the backend and the chain disagree, the chain wins.

### What is included today

- A dependency-light Node.js + TypeScript HTTP server (`node:http`, ESM) that listens on `PORT` (default `8787`).
- `GET /health` → `{"ok":true,"service":"remitloop-backend"}` for uptime checks.
- `GET /network` → the Stellar network passphrase the service is configured for (currently Testnet).
- Any other route → `404 {"error":"not_found"}`.
- `@stellar/stellar-sdk` 17.2.1 wired in, ready for RPC calls and contract-event indexing.
- A baseline test (`node --test`) so CI has a harness to grow from.

### Tech stack

Node.js · TypeScript 5.8 · `tsx` · `@stellar/stellar-sdk` 17.2.1

### Getting started

```bash
npm install
cp .env.example .env   # then fill in the values below
npm run dev           # start the API with tsx
npm run build         # type-check and compile with tsc
npm test              # run the test suite
```

Quick check once it is running:

```bash
curl http://localhost:8787/health
```

### Configuration

| Variable | Purpose | Default |
|---|---|---|
| `PORT` | Port the HTTP server listens on | `8787` |
| `STELLAR_RPC_URL` | Soroban RPC endpoint to read chain state from | `https://soroban-testnet.stellar.org` |
| `DATABASE_URL` | Database for indexed data / read models | *(empty — set when persistence is added)* |
| `CONTRACT_ID` | Deployed RemitLoop contract to index | *(empty — set after deployment)* |

## Roadmap

- Subscribe to RemitLoop contract events through Stellar RPC and index them.
- Add persistence behind `DATABASE_URL` and domain read endpoints for the app.
- Add structured logging, rate limiting and real tests beyond the baseline.

## Maintainer

`@ollypee22`

## Status

**v0.1.0 development baseline — not audited and not production-ready.**

## Stellar alignment

The project uses Stellar/Soroban where on-chain state is the source of truth or where deterministic settlement is valuable. Off-chain services are kept out of consensus-critical logic.

## License

Apache-2.0
