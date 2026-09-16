# ImpactFlow

> **Fund projects. Verify progress. Release funds transparently.**

ImpactFlow is an open-source, milestone-based funding platform built on [Stellar](https://stellar.org/) and [Soroban](https://soroban.stellar.org/). It connects project creators with contributors through a transparent, trustless system that releases funds only when independently verified milestones are completed.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Built on Stellar](https://img.shields.io/badge/Built%20on-Stellar%2FSoroban-7B61FF)](https://stellar.org)
[![Status: Foundation](https://img.shields.io/badge/Status-Foundation-yellow)]()

---

## Table of Contents

- [The Problem](#the-problem)
- [The Solution](#the-solution)
- [How the Platform Works](#how-the-platform-works)
- [Project Lifecycle](#project-lifecycle)
- [Milestone Lifecycle](#milestone-lifecycle)
- [Architecture](#architecture)
- [Frontend](#frontend)
- [Backend](#backend)
- [Smart Contract](#smart-contract)
- [Why Stellar / Soroban](#why-stellar--soroban)
- [Security Considerations](#security-considerations)
- [Development Roadmap](#development-roadmap)
- [Contributing](#contributing)
- [License](#license)

---

## The Problem

Traditional crowdfunding platforms suffer from a fundamental trust deficit:

- **No accountability after funding.** Once contributors send funds, they have no mechanism to enforce delivery. Creators can miss milestones, go silent, or abandon projects entirely.
- **Opaque fund management.** Contributors cannot see how funds are held, spent, or distributed.
- **Centralised gatekeeping.** Platform operators act as intermediaries who control fund release, dispute resolution, and platform access — introducing single points of failure and potential bias.
- **All-or-nothing inefficiency.** If a project partially succeeds, contributors have no way to recover funds for the undelivered portion.
- **No on-chain verification.** Evidence of milestone completion lives off-chain, unauditable and easily manipulated.

---

## The Solution

ImpactFlow replaces trust with transparency by encoding funding rules directly into a Soroban smart contract on the Stellar network.

- **Milestone-gated fund release.** Contributor funds are locked in the smart contract and released incrementally — only when a designated verifier independently confirms that a milestone has been completed.
- **No custodians.** No platform operator ever holds or controls contributor funds. The contract enforces the rules autonomously.
- **Partial refunds.** If a project fails to complete all milestones, contributors can reclaim the proportional share of funds allocated to uncompleted milestones.
- **On-chain auditability.** Every project creation, funding action, milestone approval, and fund release is recorded as an immutable event on the Stellar blockchain.
- **Permissionless participation.** Anyone with a Stellar wallet can create a project, contribute to one, or be assigned as a verifier.

---

## How the Platform Works

1. **Creator** defines a project with a title, description, funding goal, deadline, and an ordered list of milestones. Each milestone specifies its deliverable description and the percentage of total funds it represents.
2. **Creator** assigns a **Verifier** — an independent address responsible for confirming milestone completion. The verifier may be a trusted individual, a DAO, a multisig wallet, or any Stellar account.
3. **Contributors** browse projects and fund any project they believe in. Funds are sent directly to the smart contract and locked until milestone conditions are met.
4. **Creator** works toward the first milestone and, upon completion, submits evidence off-chain (e.g., a GitHub commit, a document, a demo link) to the platform backend.
5. **Verifier** reviews the evidence and, if satisfied, calls the smart contract`s `approve_milestone` function.
6. **Smart contract** releases the portion of funds allocated to that milestone directly to the creator`s wallet.
7. Steps 4–6 repeat for each subsequent milestone.
8. If the project is abandoned or a milestone is rejected and the project closed, contributors may call `claim_refund` for any remaining locked funds.

---

## Project Lifecycle

```
DRAFT -> OPEN -> FUNDED -> IN_PROGRESS -> COMPLETED
                                       \-> FAILED
```

| State         | Description                                                                  |
|---------------|------------------------------------------------------------------------------|
| `DRAFT`       | Project has been submitted but not yet published to the contract.            |
| `OPEN`        | Project is published on-chain and accepting contributions.                   |
| `FUNDED`      | The funding goal has been reached; no further contributions are accepted.    |
| `IN_PROGRESS` | Milestones are being executed and verified.                                  |
| `COMPLETED`   | All milestones have been verified and all funds disbursed to the creator.    |
| `FAILED`      | The project did not reach its goal by the deadline, or was closed early. Contributors may claim refunds. |

---

## Milestone Lifecycle

```
PENDING -> SUBMITTED -> APPROVED -> FUNDS_RELEASED
                     \-> REJECTED -> (creator may resubmit)
```

| State            | Description                                                                |
|------------------|----------------------------------------------------------------------------|
| `PENDING`        | Milestone has not yet been submitted by the creator.                       |
| `SUBMITTED`      | Creator has submitted evidence of completion; awaiting verifier review.    |
| `APPROVED`       | Verifier has confirmed completion; fund release is authorised.             |
| `REJECTED`       | Verifier has rejected the submission; creator may address feedback and resubmit. |
| `FUNDS_RELEASED` | The milestone`s allocated funds have been transferred to the creator.      |

---

## Architecture

ImpactFlow is composed of three independent, loosely coupled components:

```
+----------------------------------------------------------+
|                      ImpactFlow                          |
|                                                          |
|  +--------------+   REST/WS    +------------------+     |
|  |   Frontend   |<------------>|     Backend      |     |
|  |  React/Vite  |              |  Node.js / TS    |     |
|  +------+-------+              +--------+---------+     |
|         |  Stellar SDK                  |  Stellar SDK  |
|         |  (wallet signing)             |  (indexing)   |
|         v                               v               |
|  +------------------------------------------------------+|
|  |              Stellar / Soroban                       ||
|  |          ImpactFlow Smart Contract                   ||
|  +------------------------------------------------------+|
+----------------------------------------------------------+
```

All state that governs money — funding, milestone approval, and refunds — lives exclusively in the smart contract. The backend is a convenience and indexing layer; the frontend is a presentation layer. Neither is required for the contract to function correctly.

---

## Frontend

**Directory:** `Frontend/`

The frontend is a React + TypeScript + Vite application styled with Tailwind CSS.

### Responsibilities

- **Project Discovery** — Browse, search, and filter published projects.
- **Wallet Connection** — Connect a Stellar-compatible wallet (e.g., Freighter) to sign transactions.
- **Project Creation** — Guided form for creators to define project metadata, milestones, and verifier address.
- **Funding** — Allow contributors to fund open projects directly from their wallet.
- **Milestone Tracking** — Display real-time milestone progress and verifier status for any project.
- **Creator Dashboard** — Allow creators to submit milestone evidence and monitor project health.
- **Contributor Dashboard** — Show a contributor`s funded projects, milestone progress, and refund eligibility.
- **Verifier Dashboard** — Allow verifiers to review submitted evidence and approve or reject milestones.

See `Frontend/README.md` for full details.

---

## Backend

**Directory:** `Backend/`

The backend is a Node.js + TypeScript REST API.

### Responsibilities

- **Off-chain metadata storage** — Store project titles, descriptions, images, and milestone descriptions in a database.
- **Milestone evidence management** — Receive, store, and serve creator-submitted milestone evidence (links, documents, notes).
- **API endpoints** — Expose a clean REST API consumed by the frontend.
- **Stellar event indexing** — Monitor the Soroban contract for on-chain events and keep the database in sync.
- **Search and filtering** — Enable efficient project discovery beyond what on-chain queries can support.

See `Backend/README.md` for full details.

---

## Smart Contract

**Directory:** `smart contract/`

The smart contract is written in Rust and deployed to Stellar`s Soroban VM.

### Responsibilities

- **Project registration** — Record the creator address, funding goal, deadline, verifier address, and milestone structure on-chain.
- **Contribution management** — Accept XLM or Soroban token contributions and track per-contributor balances.
- **Funding goal enforcement** — Prevent milestone execution until the funding goal is reached.
- **Verifier-gated milestone approval** — Only the designated verifier may approve a milestone.
- **Incremental fund release** — Release only the funds allocated to an approved milestone; keep remaining funds locked.
- **Contributor refunds** — Allow contributors to reclaim their proportional share of locked funds if the project fails.

See `smart contract/README.md` for full details.

---

## Why Stellar / Soroban

| Property                    | Why It Matters for ImpactFlow                                       |
|-----------------------------|---------------------------------------------------------------------|
| **Low transaction fees**    | Micro-contributions remain economically viable.                    |
| **Fast finality (~5 s)**    | Milestone approvals and fund releases confirm quickly.             |
| **Soroban smart contracts** | Expressive, auditable Rust contracts with deterministic execution.  |
| **Built-in asset model**    | Native support for custom tokens alongside XLM.                    |
| **Global accessibility**    | Stellar`s network is open to anyone worldwide without KYC.         |
| **Eco-friendly consensus**  | Stellar`s SCP is energy-efficient compared to proof-of-work chains. |

---

## Security Considerations

- **Contract-enforced rules.** Fund release logic is encoded in the smart contract and cannot be altered by any off-chain party, including the ImpactFlow team.
- **Verifier independence.** The verifier is set at project creation and cannot be changed unilaterally by the creator.
- **No admin keys.** The deployed contract has no privileged owner function that can drain funds or override milestone decisions.
- **Off-chain evidence is advisory.** The backend stores evidence for transparency, but the contract does not depend on the backend for correctness. If the backend goes offline, funds remain safe.
- **Input validation.** All contract entry points validate inputs (milestone percentages must sum to 100%, deadlines must be in the future, etc.).
- **Audit readiness.** The contract will be written with clarity and full documentation to facilitate third-party security audits before mainnet deployment.
- **Frontend security.** The frontend never holds private keys. All signing is delegated to the user`s wallet extension (e.g., Freighter).

See `SECURITY.md` for vulnerability reporting procedures.

---

## Development Roadmap

### Phase 0 — Foundation *(current)*
- [x] Repository structure and documentation
- [ ] Development environment setup
- [ ] Dependency installation

### Phase 1 — Smart Contract
- [ ] Core data structures (Project, Milestone, Contribution)
- [ ] Project creation and registration
- [ ] Contribution acceptance and tracking
- [ ] Verifier milestone approval
- [ ] Incremental fund release
- [ ] Contributor refund mechanism
- [ ] Contract unit and integration tests
- [ ] Testnet deployment

### Phase 2 — Backend
- [ ] Project and milestone API (CRUD)
- [ ] Milestone evidence submission endpoint
- [ ] Stellar event indexer (Horizon / Soroban RPC)
- [ ] Database schema and migrations
- [ ] Authentication (wallet-signed messages)
- [ ] API documentation

### Phase 3 — Frontend
- [ ] Design system and component library
- [ ] Wallet connection (Freighter)
- [ ] Project discovery page
- [ ] Project creation flow
- [ ] Funding flow
- [ ] Milestone tracking view
- [ ] Creator, contributor, and verifier dashboards

### Phase 4 — Integration and Testing
- [ ] End-to-end integration tests
- [ ] Testnet user testing
- [ ] Security audit
- [ ] Performance optimisation

### Phase 5 — Mainnet Launch
- [ ] Mainnet contract deployment
- [ ] Production infrastructure
- [ ] Public launch

---

## Contributing

We welcome contributions from developers, designers, writers, and domain experts.

Please read `CONTRIBUTING.md` for:
- How to report bugs and request features
- Branch naming and commit conventions
- Pull request process
- Code style guidelines

All contributors are expected to follow `CODE_OF_CONDUCT.md`.

---

## License

ImpactFlow is released under the [MIT License](LICENSE).

Copyright (c) 2026 ImpactFlow Contributors

