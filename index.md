---
okf_version: '0.2'
title: Tidewell Insurance - Claims Domain Architecture
description: Welcome to the org-level architecture knowledge base for the Claims Domain at Tidewell Insurance.
generated:
  at: '2026-10-05T15:54:09Z'
---

# Tidewell Insurance - Claims Domain Architecture

Welcome to the org-level architecture knowledge base for the **Claims Domain** at Tidewell Insurance. This documentation synthesises service interactions, event-driven workflows, shared infrastructure, and architectural boundaries across our claim handling ecosystem.

---

## System Ecosystem & Service Landscape

```
                                  +-----------------------+
                                  |     claims-intake     |
                                  +-----------------------+
                                              |
                                claims.claim.reported
                                              |
                       +----------------------+----------------------+
                       |                                             |
                       v                                             v
             +-------------------+                         +-------------------+
             |   fraud-scoring   | -- fraud.score.flagged --> | claims-management |
             +-------------------+                         +-------------------+
                       |                                             |         |
         POST /v1/scores (sync fallback)                             |         |
                       +---------------------------------------------+         |
                                                                               |
                                 +---------------------------------------------+
                                 |                      |
                       claims.claim.settled      POST /v1/payouts
                                 |                      |
                                 v                      v
                       +-------------------+  +-------------------+
                       | notifications-hub |  | payments-gateway  |
                       +-------------------+  +-------------------+
                                                        |
                                               payments.payout.sent
                                                        |
                                                        v
                                              +-------------------+
                                              | claims-management |
                                              +-------------------+
```

---

## Core Services

| Service | Tech Stack | Ownership | Role / Responsibilities |
| :--- | :--- | :--- | :--- |
| [[ap:fraud-scoring/index#fraud-scoring|fraud-scoring]] | Python 3.11 / FastAPI | Claims / Handling Squad (`#tw-claims`) | Evaluates incoming claims for fraud risk (heuristics + vendor), publishes `fraud.score.flagged` on high risk (> 0.8). |
| [[ap:claims-management/index#claims-management|claims-management]] | Java / Spring Boot | Claims / Handling Squad (`#tw-claims`) | Core orchestrator for the claim lifecycle: handler assignment, coverage verification, settlement, and payouts. |

### Upstream & Downstream Dependencies
- **`claims-intake`**: Upstream service initiating claim workflows via `claims.claim.reported`.
- **`policy-admin`**: Policy lifecycle service; currently subjected to direct database queries on `policy_cover` by `claims-management`.
- **`payments-gateway`**: Disburses settled claims via `POST /v1/payouts` and emits `payments.payout.sent`.
- **`notifications-hub`**: Customer notification router triggered by `claims.claim.settled`.
- **External Fraud Vendor**: Third-party API queried by `fraud-scoring` to evaluate customer risk profiles.

---

## Domain Architecture Topics

- **[[architecture/claims-domain|Claims Domain Workflows]]**: Detailed end-to-end claim lifecycle from intake to settlement and payout.
- **[[architecture/events|Event Choreography & AsyncAPI]]**: Unified event catalog across GCP Pub/Sub topics.
- **[[architecture/infrastructure-and-debt|Infrastructure & Technical Debt]]**: Shared infrastructure, protocol contracts, and database boundary coupling.

<!-- okf:contents -->

## Contents

- [Architecture](/architecture/index.md) — 3 pages.
- [Change log](/log.md) — every generation and sync, newest first.

<!-- /okf:contents -->
