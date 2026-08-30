# Index: queue name → owning consumer

> **Illustrative example.** Every queue, service and incident named here is fictional, written
> to show the intended shape and density of an index. Nothing here describes real
> infrastructure. Delete this file once your own indexes make it redundant.

You are looking at a queue with a growing backlog, or a monitoring page listing queues by depth,
and you have a name like `orderflow.authorizations.pending`. This index says who is supposed to
be draining it and what it means when nobody is.

Rows point; they do not explain. Service facts live in
[`../../systems/docs/`](../../systems/docs/) and fixes live in runbooks.

Sorted alphabetically by queue name - you always arrive with the exact string, so scanning beats
any cleverer order.

---

| Queue | Consumer | Normal depth | Backlog means | Safe to purge? |
|---|---|---|---|---|
| `orderflow.authorizations.pending` | [`orderflow`](../../systems/docs/_EXAMPLE.md) | < 50, spikes to ~2k at 09:00 | `payments` is slow or down; orderflow is healthy and holding work deliberately | **No.** Each message is an order awaiting authorization; purging loses paid-for orders |
| `notifier.outbound.email` | `notifier` | < 200 | Either the email vendor is rate-limiting, or the consumer is stopped. Check consumption rate before assuming failure - a slow consumer looks identical from here | Only messages older than 48h, and only with the comms team's agreement |
| `analytics.events.raw` | `analytics` | < 100k, drains nightly | Almost always benign - it fills all day by design. Only actionable if it is still growing after the 02:00 drain | Yes. Reconstructible from `orders-db` |
| `fulfilment.picks.retry` | `fulfilment` | 0 | Something is failing repeatedly. Depth here is a direct count of stuck shipments | **No.** Escalate to the Fulfilment team |

---

## Column definitions

**Normal depth** is the column that does the work. Without it, every non-zero depth looks like
an incident and someone investigates `analytics.events.raw` at 3am for behaving exactly as
designed. Give a typical range and name the expected spikes with their times.

**Backlog means** is written from the position of someone who does not know the system: what
the depth is evidence *of*. "Messages are queued" is not an answer; "payments is slow and
orderflow is deliberately holding work" is.

**Safe to purge** is a yes/no with the consequence stated. Under pressure, purging looks like an
easy win, and the cost of getting it wrong is unrecoverable. A bare "no" without the reason gets
overridden by someone who thinks they know better.

## Filling this in

Queue definitions come from the infrastructure config in `infra/queues/`, so the name and
consumer columns can be reconciled against it - `make check-queue-index` fails CI when they
diverge. Normal depth comes from the last 30 days of metrics and is worth refreshing after any
significant traffic change; the other two columns are written from experience and cannot be
generated.

## Maintenance

Add a row when a queue is created, as part of the same change - the CI check enforces it. Refine
"normal depth" and "backlog means" after any incident that involved the queue, while the context
is still fresh.
