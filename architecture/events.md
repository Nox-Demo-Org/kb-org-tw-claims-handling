---
type: Page
title: Event Choreography & Specifications
description: AsyncAPI 2.6.0 specification across the claims event stream running on Google Cloud Pub/Sub.
resource: https://github.com/Nox-Demo-Org/kb-org-tw-claims-handling/blob/main/architecture/events.md
tags:
- org-tw-claims-handling
- architecture
generated:
  by: nox/gemini-3.7-flash
  at: '2026-10-05T15:54:09Z'
---

# Event Choreography & Specifications

AsyncAPI 2.6.0 specification across the claims event stream running on Google Cloud Pub/Sub.

---

## Topic & Event Registry

| Event Name | Producer | Consumers | Description |
| :--- | :--- | :--- | :--- |
| `claims.claim.reported` | `claims-intake` | [[ap:claims-management/index#claims-management|claims-management]], [[ap:fraud-scoring/index#fraud-scoring|fraud-scoring]] | Emitted when a new claim is logged. |
| `claims.handler.assigned` | [[ap:claims-management/index#claims-management|claims-management]] | Downstream hubs | Emitted when a handler is assigned to an active claim. |
| `fraud.score.flagged` | [[ap:fraud-scoring/index#fraud-scoring|fraud-scoring]] | [[ap:claims-management/index#claims-management|claims-management]], Counter-Fraud | Published when fraud score > 0.8. |
| `claims.claim.settled` | [[ap:claims-management/index#claims-management|claims-management]] | `payments-gateway`, `notifications-hub` | Emitted when settlement terms are finalized. |
| `payments.payout.sent` | `payments-gateway` | [[ap:claims-management/index#claims-management|claims-management]] | Confirms disbursement of settlement funds. |

---

## Event Payloads

### `claims.claim.reported`
```json
{
  "claim_id": "CLM-10023",
  "policy_id": "POL-99212",
  "customer_id": "CUST-4412",
  "peril": "theft",
  "incident_date": "2026-03-01T08:00:00Z",
  "description": "Stolen equipment",
  "excess_amount": 25000
}
```

### `fraud.score.flagged`
```json
{
  "claim_id": "CLM-10023",
  "score": 0.82,
  "reasons": []
}
```

### `claims.handler.assigned`
```json
{
  "claim_id": "CLM-10293",
  "policy_id": "POL-99201",
  "customer_id": "CUST-4412",
  "handler_name": "Jane Doe",
  "assigned_at": "2026-03-01T14:22:00Z"
}
```

### `claims.claim.settled`
```json
{
  "claim_id": "CLM-10293",
  "amount_pence": 150000,
  "settled_at": "2026-03-02T11:00:00Z"
}
```