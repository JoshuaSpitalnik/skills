# Data shapes

What each class of observability tool gives back, described by shape rather than by vendor
format. Knowing the shape before you have the data lets you plan an investigation, and lets a
script or an agent reason about a result without a worked example of every vendor's JSON.

Vendors differ in field names and encoding. They do not differ much in shape: a metric is a
series of numbers over time with labels, whatever the product is called.

---

## Metric series

A named measurement, a set of labels identifying the dimension, and points over time.

```
{
  "name": "http.request.duration",
  "labels": { "service": "...", "env": "prod", "endpoint": "...", "status": "500" },
  "unit": "milliseconds",
  "aggregation": "p99",          // raw | sum | rate | avg | p50 | p95 | p99 | max
  "resolution_seconds": 60,
  "points": [ { "t": "2026-03-14T09:00:00Z", "v": 143.2 }, ... ]
}
```

**What to watch for.** The aggregation is part of the meaning: a p99 and an average of the same
measurement tell you different things, and confusing them leads to "it looks fine" while a
quarter of users are timing out. Resolution hides spikes shorter than one bucket, so a
one-minute series cannot show a fifteen-second outage. A gap in `points` might be no traffic or
no collection - those are very different and usually indistinguishable from the series alone.
Counters reset when a process restarts, so a rate calculated across a restart shows a spurious
dip or spike.

<!-- FILL: your metric naming scheme, the labels available on every metric, and your default
     resolution and retention. -->

---

## Log event

One structured record. The fields required by
[`../../conventions-writing/patterns/conventions.md`](../../conventions-writing/patterns/conventions.md#logging)
are what make these searchable and joinable.

```
{
  "timestamp": "2026-03-14T09:07:41.221Z",
  "level": "error",
  "service": "...",              // matches the name in ../../systems/docs/
  "env": "prod",
  "trace_id": "...",             // the join key to traces and to other services' logs
  "event": "payment.capture.failed",   // stable slug - what by-error.md indexes on
  "error_code": "...",
  "message": "...",              // human-readable, may change, do not parse
  "...": "..."                   // context fields specific to the event
}
```

**What to watch for.** Query results are usually truncated and often silently - a result showing
100 lines may be the first 100 of a million, which makes "I only see a few" a dangerous
conclusion. Ordering is by ingestion, which is not emission: lines can arrive out of order and
late. Multi-line entries such as stack traces may be split into separate records by the
collector. Retention is typically the shortest of any tool here, so old incidents may have no
logs left at all.

<!-- FILL: your retention window, result limits, and whether you have a way to export more than
     the UI shows. -->

---

## Trace

One request's path, as a tree of spans.

```
{
  "trace_id": "...",
  "root_service": "...",
  "duration_ms": 2431,
  "spans": [
    {
      "span_id": "...", "parent_span_id": null,
      "service": "...", "operation": "POST /v1/orders",
      "start": "...", "duration_ms": 2431,
      "status": "error",
      "attributes": { "http.status_code": 504, "...": "..." }
    }
  ]
}
```

**What to watch for.** Traces are sampled, so the specific request you care about may not exist -
and error and success paths are often sampled at different rates. A span's duration includes its
children, so the time actually spent *in* a service is its duration minus its children's; reading
the raw number is how a fast service gets blamed for a slow dependency. A missing span may mean
an uninstrumented service rather than a skipped call, which makes an unmonitored hop invisible
rather than obviously absent. Clock skew between hosts can make a child appear to start before
its parent.

<!-- FILL: your sampling rate, which services are instrumented, and - importantly - which are
     not. -->

---

## Alert / incident record

See [`../guides/field-reference.md`](../guides/field-reference.md) for what each field means and
what it should change about your next action.

```
{
  "alert_name": "...", "severity": "...", "service": "...", "env": "prod",
  "fired_at": "...", "resolved_at": null,
  "threshold": 2000, "observed": 4310,
  "labels": { "region": "...", "tenant": "..." },
  "runbook_url": "..."
}
```

**What to watch for.** `fired_at` is when the condition had held for the full evaluation window,
so the problem started earlier - subtract the window before correlating with deploys. The
`service` field names the service that noticed, which is frequently not the one that is broken;
see [`../guides/nested.md`](../guides/nested.md).

---

## Deploy / change record

```
{
  "service": "...", "version": "...", "previous_version": "...",
  "started_at": "...", "completed_at": "...",
  "status": "succeeded",
  "includes_migration": false,
  "actor": "...", "change_url": "..."
}
```

**What to watch for.** Correlate against `started_at`, not `completed_at` - a rolling deploy is
serving new code from its first minute, so the impact begins well before completion. A deploy
recorded as failed may still have partially rolled out, which produces intermittent symptoms
that look nothing like a normal bad release. And `includes_migration` decides whether a rollback
is safe, which is the question you will be asked within the next two minutes.

---

## Cross-tool joins

| To go from | To | Join on |
|---|---|---|
| A log line | The full request | `trace_id` |
| A trace | Why a span failed | `trace_id` + `service` in logs |
| A metric spike | The requests behind it | Same time window + labels, then sample traces |
| An alert | The change that caused it | `fired_at` minus the evaluation window, against deploy history |
| An error signature | Prior occurrences | [`../indexes/by-error.md`](../indexes/by-error.md) |

`trace_id` is the only real join key. Everything else is correlation by time and label, which is
suggestive rather than conclusive - worth stating explicitly when you report a finding based on
it.
