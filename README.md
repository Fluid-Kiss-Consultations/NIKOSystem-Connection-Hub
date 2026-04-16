# NIKOSystem Connection Hub

> **Nascent Integration of Kernelized Optimization — Unified Product v1**

[![Contracts CI](https://github.com/Fluid-Kiss-Consultations/NIKOSystem-Diamond-v1/actions/workflows/contracts-ci.yml/badge.svg)](https://github.com/Fluid-Kiss-Consultations/NIKOSystem-Diamond-v1/actions/workflows/contracts-ci.yml)
[![Solidity](https://img.shields.io/badge/Solidity-0.8.24-363636?logo=solidity)](https://soliditylang.org/)
[![EIP-2535](https://img.shields.io/badge/EIP--2535-Diamond%20Standard-blue)](https://eips.ethereum.org/EIPS/eip-2535)
[![Foundry](https://img.shields.io/badge/Built%20with-Foundry-FFDB1C?logo=ethereum)](https://getfoundry.sh/)
[![Tests](https://img.shields.io/badge/tests-521%20passing-brightgreen)](#test-status)
[![License](https://img.shields.io/badge/license-Proprietary-red)](#license)

© 2025 John A. Welch (Founder/Systems Oracle), Fluid Kiss Consultations. All rights reserved.

---

## Overview

NIKOSystem Diamond is a modular blockchain SaaS platform built on the Diamond Standard (EIP-2535), targeting Optimism L2. It provides upgradeable on-chain logic, a strategic microkernel backend, and a framework-agnostic agent integration surface — designed so any reasoning model can connect to any agent through typed, composable stubs.

The system supports client-specific contract forks from a single diamond deployment, proven in production through the Nifty Mints implementation.

---

## Architecture

```text
NIKOSystem-Diamond-v1/
├── packages/
│   ├── contracts/    @niko/contracts  — EIP-2535 facets, libraries, Foundry tests
│   ├── backend/      @niko/backend   — Strategic microkernel, event bus, APIs
│   ├── brandi/       @niko/brandi    — Agent framework (separate product, licensed use)
│   └── frontend/     React 19 + Vite — Admin dashboard (in development)
├── .github/workflows/   CI/CD pipelines
├── turbo.json           Turborepo build pipeline
└── package.json         Workspace root (pnpm)
```

### Build Dependencies

```text
@niko/contracts (forge build)
    ↓ ABI artifacts
@niko/backend (nest build) ← depends on contracts
    ↓ API types + event bus
@niko/brandi (tsc -b) ← depends on contracts + backend
```

---

## Diamond Facets

Ten facets compose the on-chain logic layer. Each facet owns its storage namespace via the Diamond Storage pattern, preventing collision across upgrades.

| Facet | Purpose | Library |
| ------- | --------- | --------- |
| DiamondCut | Upgrade mechanism (add/replace/remove) | LibDiamond |
| DiamondLoupe | Introspection (ERC-165) | LibDiamond |
| Security | RBAC, pause, blacklist | LibSecurity |
| Monitoring | Events, health, metrics | LibMonitoring |
| Orchestrator | Agent registration and lifecycle | LibOrchestrator |
| Kernel | Cache layer with TTL and optimization | LibKernel |
| Consensus | Block proposal, validation, finalization | LibConsensus |
| Governance | Proposals, voting, execution | LibGovernance |
| Treasury | Deposits, revenue splits, distribution | LibTreasury |
| Oracle | Dual-chain aggregation, consensus verification | LibOracle |

All access control is function-scoped through `LibSecurity` — roles bind to operations, not entities.

---

## Strategic Microkernel

The backend is not a standard NestJS application. It hosts a **strategic microkernel** where `KernelService` owns the system's reactive state through an RxJS `BehaviorSubject<KernelState>` with Immer-based immutable mutations, and a `Subject<KernelEvent>` event bus. All inter-module communication routes exclusively through the event bus.

### Business Modules

Every module extends `NikoModule`, an abstract base class enforcing `shouldHandle` and `handleEvent` — guaranteeing that modules cannot bypass the kernel or call each other directly.

| Module | Responsibility |
| -------- | --------------- |
| Client | Registration, lifecycle, configuration |
| Oracle | Dual-chain aggregation, consensus verification |
| Chain | Per-client fork chains + shared main chain |
| Agent | Template-based deployment, execution tracking |
| Session | API key auth, token management, rate limiting |
| Treasury | Revenue recording, per-client accounting |
| Metrics | Prometheus registry, system observability |

---

## Agent Integration

NIKOSystem is offered as a solution for linking your agent to your reasoning model — framework-agnostic by design.

The agent integration surface exposes typed stubs that define the boundary between NIKOSystem's orchestration layer and any external agent or reasoning framework. These stubs are composable: an agent connects through event contracts and bridge mappings without coupling to NIKOSystem internals.

This means:

- **BRANDI** connects as a first-party agent through these stubs while retaining full separate-product status
- **Any third-party agent framework** can implement the same typed interface to participate in NIKOSystem orchestration
- **Any reasoning model** (local, hosted, hybrid) can be routed through the agent layer without architectural lock-in

The coupling is deliberate and minimal — enough structure to guarantee event-level validation, loose enough that the agent owns its own reasoning.

---

## Technology Stack

| Layer | Built With |
| ------- | ----------- |
| Smart Contracts | Solidity 0.8.24, Foundry |
| Contract Standard | EIP-2535 Diamond Standard |
| Target Chain | Optimism L2 |
| Backend Framework | NestJS, RxJS |
| Reactive State | Immer (immutable mutations) |
| Database | PostgreSQL |
| Cache | Redis |
| Frontend | React 19, Vite |
| Monorepo | Turborepo, pnpm workspaces |
| CI/CD | GitHub Actions |
| Agent Framework | BRANDI (licensed, separate product) |

---

## Build & Test

```bash
# Install dependencies
pnpm install

# Build all packages (dependency-aware)
turbo run build

# Run contract tests
cd packages/contracts && forge test -vvv

# Gas report
forge test --gas-report

# Run backend tests
cd packages/backend && pnpm test
```

### Test Status

**521 tests total**, all passing: 130 contracts + 304 backend + 87 frontend.

**Contracts — 130 tests across 10 suites:**

| Suite | Tests | Coverage |
| ------- | ------- | ---------- |
| Phase 0 — Security/Monitoring | 25 | RBAC, pause, blacklist, events, health |
| Phase 1 — Orchestrator | 16 | Agent lifecycle, actions, metrics |
| Phase 2 — Kernel | 13 | Cache CRUD, TTL, batch, optimization |
| Phase 3 — Consensus | 16 | Block proposal, validation, finalization, FSM |
| Phase 4 — Governance | 12 | Proposals, voting, execution, pause |
| Phase 5 — Treasury | 12 | Deposits, splits, distribution, emergency |
| Phase 6 — Oracle | 12 | Chain registration, aggregation, consensus, commit |
| Diamond | 10 | Deployment, introspection, cuts, fallback |
| Gas Efficiency | 13 | Regression guards at 1.25× measured baseline |

**Backend — 304 tests** covering microkernel state management, event bus routing, module isolation, PG write-through, and bridge connectivity.

| Category | Suites | Tests |
| ---------- | -------- | ------- |
| Kernel (service, events, state, contracts, modules) | 5 | 43 |
| Business modules (client, agent, chain, oracle, session, treasury) | 6 | 101 |
| Cascade integration (phase3, phase4, client-chain) | 3 | 24 |
| API controllers + gateway | 6 | 29 |
| Diamond (client, bridges) | 2 | 27 |
| Infrastructure (database, cache, config, snapshot) | 4 | 54 |
| Metrics | 1 | 11 |
| BRANDI stubs | 1 | 15 |

**Frontend — 87 tests** covering API client, hooks, components, and all 11 admin dashboard pages.

| Category | Suites | Tests |
| ---------- | -------- | ------- |
| API client | 1 | 6 |
| Hooks (useAuth, useDiamond) | 2 | 14 |
| Components (StatCard, Layout) | 2 | 7 |
| Pages (11 admin pages) | 12 | 55 |
| App root | 1 | 5 |

---

## Related Projects

### BRANDI

>**Branching Recursive Asynchronous Nodal Decentralized Intelligence**

BRANDI is an agentic component within NIKOSystem Diamond (`packages/brandi`) while retaining separate product status — licensed for use by NIKOSystem, not owned by it. BRANDI maintains its own identity, release cycle, and licensing.

© 2026 John A. Welch, Fluid Kiss Consultations. All rights reserved.

→ [github.com/Fluid-Kiss-Consultations/BRANDI](https://github.com/Fluid-Kiss-Consultations/BRANDI)

### Blinded Eye Foundation

NIKOSystem Diamond is licensed to the Blinded Eye Foundation for the SHANNON use case under alignment research.

The Foundation operates on a transparency principle: **high visibility signals alignment is true; low visibility signals extractive corporate interest.** If you can see the work, the work is honest.

→ [github.com/Fluid-Kiss-Consultations/Blinded-Eye-Foundation](https://github.com/Fluid-Kiss-Consultations/Blinded-Eye-Foundation)

### Nifty Mints

Client fork proof-of-concept — an audio-centric NFT marketplace demonstrating NIKOSystem's fork-and-customize deployment model. Each client instance receives an autonomous contract fork with customizable facets while inheriting the core Diamond Standard infrastructure.

---

## License

**Proprietary.** © 2025 John A. Welch, Systems Oracle, Fluid Kiss Consultations. All rights reserved.

This software and associated documentation are the exclusive property of the copyright holder. No part of this repository may be reproduced, distributed, or transmitted in any form without prior written permission.

For licensing inquiries, contact Fluid Kiss Consultations.
