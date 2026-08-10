# checkout-service

> Owns the shopping cart lifecycle and coordinates checkout across payment, inventory, and shipping.

**Bounded Context:** Order Management
**Team:** Commerce Platform · #commerce-platform (Slack)
**On-call:** PagerDuty: commerce-platform-oncall

---

## Responsibilities

- Owns the lifecycle of a shopping cart from creation to checkout
- Validates cart contents against current pricing and inventory
- Orchestrates the checkout saga across payment, inventory, and shipping
- Publishes `cart.checked_out` when checkout completes

## Interfaces

### REST API

| Method | Path | Description |
| ------ | ---- | ------------ |
| GET | /carts/{id} | Fetch a cart by id |
| POST | /carts/{id}/items | Add an item to a cart |
| DELETE | /carts/{id}/items/{itemId} | Remove an item from a cart |
| POST | /carts/{id}/checkout | Start the checkout saga |

### Published Events

| Name | Schema | Broker |
| ---- | ------ | ------ |
| cart.checked_out | CartCheckedOutV1 | Kafka |
| cart.abandoned | CartAbandonedV1 | Kafka |

### Consumed Events

| Name | Source | Broker |
| ---- | ------ | ------ |
| inventory.reserved | inventory-service | Kafka |
| payment.captured | payments-service | Kafka |

---

## Data & Dependencies

### Owned Data

| Store | Type | Description |
| ----- | ---- | ------------ |
| carts-db | postgres | Cart and cart-item records |
| cart-cache | redis | Hot cart reads, 5 min TTL |

### Sync Dependencies

| Service | Type | Criticality | Reason | Endpoints | Resilience |
| ------- | ---- | ------------ | ------ | --------- | ---------- |
| payments-service | internal | hard | Authorizes and captures payment | POST /charges | retry, circuit-breaker |
| inventory-service | internal | hard | Reserves stock for cart items | POST /reservations | retry, timeout |
| aws-s3 | cloud | soft | Stores generated receipt PDFs | | timeout |

### Async Dependencies

| Service | Event | Broker |
| ------- | ----- | ------ |
| inventory-service | inventory.reserved | Kafka |
| payments-service | payment.captured | Kafka |

---

## Operations

### SLO / SLA

- **Availability:** 99.9%
- **Latency P99:** 300ms
- **Error Budget:** 43m/month
- **SLA:** Best-effort, no external SLA

### Security

- **Authentication:** OAuth2 client-credentials (internal), Clerk session (customer-facing)
- **Authorization:** RBAC via scopes
- **Scopes:** cart:read, cart:write, checkout:execute
- **Secrets:** DATABASE_URL, PAYMENTS_API_KEY
- **Secret store:** HashiCorp Vault (https://vault.internal/commerce/checkout-service)
- **Handles PII:** Yes

### Deployment

- **Platform:** kubernetes
- **Replicas:** 4
- **CPU:** 500m
- **Memory:** 512Mi
- **Env Vars:** DATABASE_URL, REDIS_URL, PAYMENTS_API_KEY, KAFKA_BROKERS

---

## Observability

### Logs

Structured JSON logs shipped to Datadog via the sidecar log forwarder.
Correlation id: `x-request-id`.

### Metrics

| Metric | Description |
| ------ | ------------ |
| `checkout_duration_seconds` | |
| `cart_items_total` | |
| `checkout_failures_total` | |

### Distributed Tracing

Distributed tracing is **enabled**.

### Dashboards

- [Checkout Overview](https://datadog.internal/dashboards/checkout-overview)
- [Cart DB Health](https://datadog.internal/dashboards/carts-db-health)

### Alerts

- Checkout error rate > 2% over 5 minutes
- Cart DB replication lag > 30s

---

## Architecture Patterns

### DDD Aggregates

#### Cart
- **Root entity:** Cart
- **Entities:** CartItem (sku, quantity, unitPrice)
- **Value objects:** Money (amount, currency)
- **Invariants:**
  - A cart cannot contain a negative quantity
  - Total is always the sum of line items
- **References:** Customer, Promotion
- **Description:** The active shopping cart for one customer session.

### CQRS

| Role | Linked Service |
| ---- | --------------- |
| writer | checkout-service |
| reader | checkout-read-service |

### Sagas

#### CheckoutSaga
- **Type:** Orchestration
- **Trigger:** `cart.checkout_requested`
- **Happy path:** ReserveInventory → ChargePayment → ConfirmOrder → Ship
- **Compensating:** ReleaseInventory → RefundPayment → CancelOrder
- **States:** `pending` · `reserved` · `charged` · `confirmed` · `failed`

---

## Runbook

**Checkout saga stuck in `reserved`**
1. Check `inventory-service` health — a stalled reservation confirm usually
   means it's failing to publish `inventory.reserved`.
2. Requeue the saga via the admin console (`/admin/sagas/{id}/retry`).
3. If stuck > 15 min, page inventory on-call.

**Elevated `checkout_failures_total`**
1. Check the Payments dashboard for authorization failures first — most
   checkout failures originate downstream in `payments-service`.
2. Check Redis (`cart-cache`) latency; a slow cache read can cause
   optimistic-lock retries to exhaust.

---

## Architecture Decision Records

### ADR-003: Use event sourcing for the cart aggregate

**Status:** accepted

We chose event sourcing over CRUD persistence for the Cart aggregate
because checkout requires a full audit trail of cart mutations for fraud
review, and because the saga's compensating actions are easier to reason
about as a replay of recorded events rather than diffed row state.

---

## Context Pack

checkout-service is the orchestrator for the checkout saga; it does not
itself hold payment or inventory logic — it only calls out to
payments-service and inventory-service and reacts to their events. When
adding a new step to checkout, extend the saga's state machine in
`CheckoutSaga`, add a compensating step, and update the `Sagas` section of
this file.

---

_Generated by [Microservice.MD™](https://microservice.md) on 2026-08-09 — MICROSERVICE.md™ is a lightweight, AI-ready documentation specification for microservice systems, created by [Koesion](https://microservice.md)._
