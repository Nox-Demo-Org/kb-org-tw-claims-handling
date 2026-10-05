---
type: Page
title: Infrastructure, Standards & Technical Debt
description: This document details the shared technology standards, integration patterns, and known technical debt across the Claims domain services.
resource: https://github.com/Nox-Demo-Org/kb-org-tw-claims-handling/blob/main/architecture/infrastructure-and-debt.md
tags:
- org-tw-claims-handling
- architecture
generated:
  by: nox/gemini-3.7-flash
  at: '2026-10-05T15:54:09Z'
---

# Infrastructure, Standards & Technical Debt

This document details the shared technology standards, integration patterns, and known technical debt across the Claims domain services.

---

## Shared Infrastructure & Frameworks

- **Messaging**: Google Cloud Pub/Sub (`google-cloud-pubsub`) across Python and Java runtimes.
- **REST APIs**: JSON over HTTP, standard REST conventions for synchronous query and execution endpoints.
- **Languages & Frameworks**:
  - Python 3.11 / FastAPI for micro-scoring services ([[ap:fraud-scoring/index#fraud-scoring|fraud-scoring]]).
  - Java / Spring Boot for transaction and domain orchestration services ([[ap:claims-management/index#claims-management|claims-management]]).

---

## Technical Debt & Architectural Inconsistencies

### 1. Direct Cross-Service Database Read (`policy-admin` DB)
- **Context**: During the November 2023 storm surge, high read volumes caused performance bottlenecks on policy lookup endpoints. `claims-management` was granted direct read access to `policy_cover` in the `policy-admin` database.
- **Risk**: Violates bounded-context encapsulation; database schema changes in `policy-admin` can break `claims-management` without compilation or contract check errors.
- **Remediation**: Migrate `CoverCheckRepository` to query policy-admin's REST/gRPC API or consume policy snapshot events via Pub/Sub.

### 2. Dual Path for Fraud Scoring (Event vs. Sync REST)
- **Context**: `fraud-scoring` supports both an asynchronous subscription to `claims.claim.reported` and a synchronous `POST /v1/scores` endpoint.
- **Resolution**: Ensure consumers default to the event-driven choreography to decouple intake from scoring latency, reserving `POST /v1/scores` strictly for on-demand re-scoring workflows in customer and internal portals.