# NIKOSystem Connection Hub

> **Nascent Integration of Kernelized Optimization -- Unified Product v1**

[![Contracts CI](https://github.com/Fluid-Kiss-Consultations/NIKOSystem-Diamond-v1/actions/workflows/contracts-ci.yml/badge.svg)](https://github.com/Fluid-Kiss-Consultations/NIKOSystem-Diamond-v1/actions/workflows/contracts-ci.yml)
[![Solidity](https://img.shields.io/badge/Solidity-0.8.24-363636?logo=solidity)](https://soliditylang.org/)
[![EIP-2535](https://img.shields.io/badge/EIP--2535-Diamond%20Standard-blue)](https://eips.ethereum.org/EIPS/eip-2535)
[![Foundry](https://img.shields.io/badge/Built%20with-Foundry-FFDB1C?logo=ethereum)](https://getfoundry.sh/)
[![Tests](https://img.shields.io/badge/tests-422%20passing-brightgreen)](#test-status)
[![License](https://img.shields.io/badge/license-Proprietary-red)](#license)

(c) 2025 John A. Welch (Founder/Systems Oracle), Fluid Kiss Consultations. All rights reserved.

---

## Overview

NIKOSystem Diamond is a Turborepo monorepo unifying four components into a blockchain SaaS platform built on the Diamond Standard (EIP-2535), targeting Optimism L2.

## Architecture

```text
NIKOSystem-Diamond/
├── packages/
│   ├── contracts/    @niko/contracts — EIP-2535 facets, libraries, Foundry tests
│   ├── backend/      @niko/backend  — NestJS microkernel, PostgreSQL, Redis, APIs
│   ├── brandi/       @niko/brandi   — agent framework, workflow engine
│   └── frontend/     React 19 + Vite — admin dashboard
├── .github/workflows/  CI/CD pipelines
├── docker-compose.yml  Local dev (postgres + redis + backend)
├── Dockerfile          Multi-stage production build
├── turbo.json          Turborepo build pipeline
└── package.json        Workspace root (pnpm)
```

## Diamond Facets

| Facet | Purpose | Library |
|-------|---------|---------|
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

## Backend Architecture

The backend is a **NestJS application hosting a legacy microkernel pattern** — not a standard NestJS app. `KernelService` owns an RxJS `BehaviorSubject<KernelState>` with Immer-based immutable mutations and a `Subject<KernelEvent>` event bus. All inter-module communication routes through the event bus (HC-1).

### Business Modules

All extend `NikoModule` (abstract base class enforcing `shouldHandle`/`handleEvent`):

| Module | Responsibility |
|--------|---------------|
| Client | Registration, lifecycle, config |
| Oracle | Dual-chain aggregation, consensus verification |
| Chain | Per-client fork chains + shared main chain |
| Agent | Template-based deployment, execution tracking |
| Session | API key auth, token management, rate limiting |
| Treasury | Revenue recording, per-client accounting |
| Metrics | Prometheus registry (imperative, not event-driven) |

### Infrastructure Layer

| Service | Role |
|---------|------|
| DatabaseService | PostgreSQL pool, migrations, graceful degradation |
| CacheService | Redis (main + pub/sub), rate limiting, distributed locks |
| AuditService | Passive eventBus$ subscriber, persists all events |
| SnapshotService | Periodic BehaviorSubject→JSONB snapshots (D3) |

### Diamond Bridges

8 bridges (Orchestrator, Oracle, Consensus, Monitoring, Treasury, Security, Kernel, Governance) connect to on-chain Diamond facets via ethers.js. All use `@Optional()` injection — the system runs fully functional without a Diamond connection (HC-2).

## Build Dependencies

```text
@niko/contracts (forge build)
    ↓ ABI artifacts
@niko/backend (nest build) ← depends on contracts
    ↓ API types + event bus
@niko/brandi (tsc -b) ← depends on contracts + backend
```

## Quick Start

```bash
# Install dependencies
pnpm install

# Build all packages (dependency-aware)
turbo run build

# Run contract tests
cd packages/contracts && forge test -vvv

# Run backend tests
cd packages/backend && pnpm test
```

### Docker (Local Development)

```bash
# Copy environment template
cp .env.example .env

# Start postgres, redis, and backend
docker compose up --build

# Verify
curl http://localhost:3000/health
curl http://localhost:3000/health/detailed
```

Services: `postgres:14-alpine` (port 5432), `redis:7-alpine` (port 6379), backend (port 3000).

## Test Status

422 tests total (130 contracts + 292 backend), all passing.

### Contracts (Foundry) — 130 tests, 10 suites

| Suite | Tests | Status |
|-------|------:|--------|
| Phase0 — Security/Monitoring | 25 | Pass |
| Phase1 — Orchestrator | 16 | Pass |
| Phase2 — Kernel | 13 | Pass |
| Phase3 — Consensus | 16 | Pass |
| Phase4 — Governance | 12 | Pass |
| Phase5 — Treasury | 12 | Pass |
| Phase6 — Oracle | 12 | Pass |
| Diamond (core) | 10 | Pass |
| GasEfficiency | 13 | Pass |
| TestFacet (helper) | 1 | Pass |

### Backend (Jest) — 292 tests, 28 suites

| Category | Suites | Tests |
|----------|-------:|------:|
| Kernel (service, events, state, contracts, modules) | 5 | 48 |
| Business modules (client, agent, chain, oracle, session, treasury) | 6 | 87 |
| Cascade integration (phase3, phase4, client-chain) | 3 | 28 |
| API controllers + gateway | 6 | 58 |
| Diamond (client, bridges) | 2 | 24 |
| Infrastructure (database, cache, config, snapshot) | 4 | 35 |
| Metrics | 1 | 8 |
| BRANDI stubs | 1 | 4 |

## CI Pipeline

Every push and PR touching `packages/contracts/` triggers:

- **Build** -- `forge build --sizes` (contract compilation + size check)
- **Test** -- `forge test -vvv` (full suite, verbose)
- **Gas Report** -- posted as PR comment on pull requests
- **Security** -- secret scanning, storage namespace collision detection
- **Formatting** -- `forge fmt --check`

## Environment Configuration

Copy `.env.example` to `.env`. Key variable groups:

| Group | Variables | Defaults |
|-------|-----------|----------|
| Server | `PORT`, `CORS_ORIGIN` | 3000, `*` |
| PostgreSQL | `DATABASE_URL`, `DATABASE_MAX_POOL` | localhost:5432/blockchain_saas, 20 |
| Redis | `REDIS_URL`, `REDIS_MAX_RETRIES` | localhost:6379, 3 |
| Security | `JWT_SECRET`, `SESSION_TIMEOUT`, `RATE_LIMIT_*` | dev defaults |
| Snapshots | `SNAPSHOT_DEBOUNCE_MS`, `SNAPSHOT_RETENTION` | 5000ms, 5 |
| Diamond | `OPTIMISM_RPC_URL`, `DIAMOND_ADDRESS`, `PRIVATE_KEY` | empty (offline mode) |

See [`.env.example`](.env.example) for the full list.

## Stack

- **Contracts:** Solidity 0.8.24, Foundry, Diamond Standard (EIP-2535)
- **Backend:** NestJS, RxJS, PostgreSQL, Redis, GraphQL, WebSocket
- **Agents:** TypeScript, AGPL-3.0
- **Infrastructure:** Docker, n8n, Nginx, Ionos
- **Chain:** Optimism Layer 2

## Reference Repos

| Repo | Role |
|------|------|
| Surety-Diamond | Compliance facet upstream source |
| Blinded-Eye-Foundation | Alignment constraints, constitutional field |

## License

Proprietary. (c) 2025 John A. Welch (Founder/CEO), Fluid Solutions (an FKC Company). All rights reserved.
