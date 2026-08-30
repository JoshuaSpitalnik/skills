# orderflow

> **Illustrative example.** `orderflow` and every service, team and number named here are
> fictional, invented to show the intended shape and density of a service doc. Nothing here
> describes real infrastructure. Delete this file once your own service docs make it redundant.

> Last reviewed: 2026-03-02 by the Fulfilment team
> Status: active

## What it owns

The lifecycle of a customer order from submission to fulfilment handoff: validating the basket,
reserving stock, requesting payment authorization, and emitting the events that downstream
services react to. It is the authority on an order's *state* - if two systems disagree about
whether an order is confirmed, orderflow is right by definition.

**It is not responsible for:** taking payment (that is `payments`, orderflow only requests
authorization and records the outcome), stock levels (`inventory` owns the counts; orderflow
holds reservations against them), or customer notifications (`notifier` subscribes to
orderflow's events and decides what to send).

## Ownership

| | |
|---|---|
| Team | Fulfilment |
| On-call rotation | fulfilment-primary |
| Escalation after hours | Fulfilment lead for SEV1 and SEV2 only; SEV3 waits for business hours |
| Source | `github.com/example/orderflow`, service code under `src/` |

## Upstream - what it depends on

| Depends on | Via | Criticality | If it is down |
|---|---|---|---|
| `payments` | sync HTTP | critical | No order can be confirmed. Submissions are accepted and held as `pending_authorization`, so the queue drains on recovery rather than losing orders |
| `inventory` | sync HTTP | critical | Stock cannot be reserved; submissions are rejected at validation with a "temporarily unavailable" message |
| `catalog` | sync HTTP, cached 15 min | degraded | Serves stale prices from cache for up to 15 minutes, then rejects new submissions. Existing in-flight orders are unaffected |
| `orders-db` (PostgreSQL) | direct | critical | Total outage; nothing can be read or written |
| `events` (Kafka) | async publish | degraded | Orders are still confirmed, but downstream services stop hearing about them - notifications and fulfilment silently stall. Publishes buffer to disk for roughly 30 minutes |

## Downstream - what depends on it

| Consumer | Via | Criticality to them | What they see if this is down |
|---|---|---|---|
| `storefront` | sync HTTP | critical | Checkout fails; browsing still works |
| `notifier` | async events | critical | No order emails go out. No error, just silence - which is why this outage is often reported by customers rather than detected |
| `fulfilment` | async events | critical | Nothing is picked or shipped; the backlog clears on recovery |
| `analytics` | nightly batch read of `orders-db` | optional | Yesterday's numbers are missing; nobody notices until the weekly report |

## Data

| Store | Contains | Retention | Notes |
|---|---|---|---|
| `orders-db` (PostgreSQL, primary + 1 replica) | Orders, line items, reservations, state transitions | 7 years (regulatory) | Not shared. Analytics reads the replica; a heavy analytics query has previously saturated it |
| `orderflow-cache` (Redis) | Catalog price cache, idempotency keys | 15 min / 24 h | Loss is survivable: prices refetch, and idempotency keys expiring early risks duplicate submissions only within the loss window |

## Entry points

- `POST /v1/orders` - customer submission, via storefront. Can be paused with the
  `orderflow.accept_submissions` flag; queued orders are unaffected.
- `POST /v1/orders/{id}/cancel` - customer or support cancellation.
- Consumer of `payments.authorization.completed` - resolves pending orders. Pausing this
  consumer is safe; the topic retains 24 hours.
- Nightly `reconcile-authorizations` job, 02:00 UTC - reconciles orders stuck in
  `pending_authorization`. Safe to skip a night; safe to run twice.

## Configuration and deploys

| | |
|---|---|
| How it deploys | Rolling, 4 replicas, via CI on merge to `main` |
| Typical deploy duration | 6-8 minutes to full rollout |
| How to roll back | `deployctl rollback orderflow --to-previous` - takes about 3 minutes. Safe unless the deploy included a migration; check the release notes first |
| Feature flags that change its behavior | `orderflow.accept_submissions`, `orderflow.strict_price_check`, `orderflow.async_authorization` |

## Known failure modes

| Failure | Looks like | Usual cause | First action |
|---|---|---|---|
| Authorization backlog | `pending_authorization` count climbing, checkout appears slow | `payments` latency, not orderflow itself | Check `payments` health before touching orderflow. Restarting orderflow makes this worse - it drops in-flight retries |
| Reservation leak | Stock unavailable while `inventory` shows units free | Cancelled orders failing to release reservations, usually after a partial deploy | Run `orderflow-release-stale-reservations`; then find out why the release path failed |
| Replica saturation | Read latency spike at ~01:00, writes unaffected | Analytics batch overlapping with the nightly job | Confirm with the analytics team before killing the query; it is idempotent and safe to kill |
| Duplicate submissions | Two identical orders seconds apart | Redis eviction dropping idempotency keys under memory pressure | Check Redis memory. The duplicates are real orders and need manual cancellation |

## Capacity and limits

Sustains roughly 400 submissions/second; measured breaking point is around 650, where the
`orders-db` connection pool (100 connections, 25 per replica) saturates. `payments` rate-limits
us to 500 authorization requests/second and returns 429 above that. Kafka publish buffer holds
about 30 minutes of events at normal volume.

## Links

- Dashboards: `grafana/orderflow-overview`, `grafana/orderflow-db`
- Alerts defined for it: `OrderflowHighLatency`, `OrderflowAuthorizationBacklog`,
  `OrderflowErrorRate`, `OrderflowDBReplicaLag` - all listed in
  [`../../triage-structure/indexes/by-alert.md`](../../triage-structure/indexes/by-alert.md)
- Runbooks: `runbooks/orderflow/`
- Design docs: ADR-014 (order state machine), ADR-022 (async authorization)
