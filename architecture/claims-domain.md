---
type: Page
title: Claims Domain Workflows & Lifecycle
description: This document outlines the end-to-end business workflows and cross-service orchestration across the claims processing domain.
resource: https://github.com/Nox-Demo-Org/kb-org-tw-claims-handling/blob/main/architecture/claims-domain.md
tags:
- org-tw-claims-handling
- architecture
generated:
  by: nox/gemini-3.7-flash
  at: '2026-10-05T15:54:09Z'
---

# Claims Domain Workflows & Lifecycle

This document outlines the end-to-end business workflows and cross-service orchestration across the claims processing domain.

---

## 1. Claim Ingestion & Fraud Assessment

1. A policyholder or agent reports a claim through `claims-intake`.
2. `claims-intake` broadcasts the `claims.claim.reported` event over Google Cloud Pub/Sub.
3. **Parallel Processing**:
   - **[[ap:fraud-scoring/index#fraud-scoring|fraud-scoring]]** consumes the event and evaluates the risk score:
     - Computes an internal heuristic score based on policy age (<30 days adds 0.4), claim frequency (>=2 claims in 3 years adds 0.3), and peril type (`theft`/`accidental_damage` add 0.1).
     - Fetches an external vendor score via HTTP.
     - Calculates composite score: `0.6 * internal + 0.4 * vendor`.
     - If composite score > `0.8`, it emits `fraud.score.flagged`.
   - **[[ap:claims-management/index#claims-management|claims-management]]** ingests `claims.claim.reported` to initialize the claim record in `submitted` status.
4. If `claims-management` receives `fraud.score.flagged`, the claim is routed to the counter-fraud team for manual review.

---

## 2. Handler Assignment & Coverage Verification

- **Handler Assignment**: `claims-management` assigns a claims handler (SLA target: 1 working day) and transitions claim status to `assigned`, emitting `claims.handler.assigned`.
- **Coverage & Excess Check**: `claims-management` verifies cover terms and policy excess. *(Note: This currently reads directly from the `policy_cover` table in the `policy-admin` database).* 

---

## 3. Settlement & Payout Processing

1. **Negotiation & Settlement**: When a claim is agreed, `claims-management` transitions status to `settled` and publishes `claims.claim.settled`.
2. **Approval Governance**:
   - Standard payouts are dispatched directly.
   - Payouts exceeding **£25,000** require dual-approval authorization within `claims-management`.
3. **Payment Initiation**: `claims-management` calls `payments-gateway` via `POST /v1/payouts` with an `idempotency_id`.
4. **Disbursement Confirmation**: `payments-gateway` processes the transfer and emits `payments.payout.sent`.
5. **Completion**: `claims-management` consumes `payments.payout.sent` and marks the claim status as `paid`.