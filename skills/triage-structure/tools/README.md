# Observability tools

Which tool answers which question. Written vendor-neutrally by tool *class*, because the classes
outlive the vendors - the reasoning below survives a migration, the product names do not.

Most wasted investigation time comes from asking the wrong class: hunting for a cause in
dashboards when you needed a trace, or reading individual log lines when you needed a rate.

## Which tool for which question

| Your question | Ask | Not |
|---|---|---|
| Is this happening, how much, and since when? | Metrics | Logs - counting log lines by eye is slow and you will get it wrong |
| Is it getting worse? | Metrics | |
| What exactly happened to *this one* request? | Traces, then logs by trace id | Metrics - an aggregate cannot tell you about one request |
| Where in the chain did the time go? | Traces | Logs - reconstructing timing from timestamps across services is guesswork |
| What was the error, in detail? | Logs, filtered by the error signature | |
| Which of these failures are the same failure? | Logs aggregated by `event` slug | Reading them individually |
| What changed? | Deploy and flag history | |
| Who is affected? | Metrics split by tenant, region, or endpoint | |
| Has this happened before? | Incident history, and [`../indexes/by-error.md`](../indexes/by-error.md) | |
| Is the dependency healthy? | Its own metrics - not your error rate calling it | Your errors show that *something* is wrong, not what |

## The classes

**Metrics** - aggregated numbers over time. Cheap to query, retained long, and the only
practical way to answer "how much" and "since when". They cannot tell you about an individual
request, and a metric that was not being collected before the incident cannot be collected
retroactively.

**Logs** - discrete events with structure. The detail layer: what the error was, what the
parameters were. Expensive to query at volume and usually retained for a shorter window than
metrics. Only as useful as the structure in them - see the logging rules in
[`../../conventions-writing/patterns/conventions.md`](../../conventions-writing/patterns/conventions.md#logging).

**Traces** - one request's path across services, with timing per hop. The right tool for "where
did the time go" and "which service actually failed", and the fastest way to disprove a theory
about which hop is slow. Usually sampled, so a specific request may simply not be there - and
error paths are often sampled at a different rate than success paths, which surprises people.

**Paging and incident records** - who was told, when, and what was done. The audit trail and the
source of prior-incident history. See [`../guides/field-reference.md`](../guides/field-reference.md).

**Deploy and change history** - what changed and when. The highest-yield first check in most
investigations, and the one most often skipped because it feels too simple.

## Reading the tools together

**Metrics to scope, traces to locate, logs to explain.** Metrics tell you how big and since
when. Traces tell you which hop. Logs tell you why that hop failed. Going straight to logs skips
the two cheaper steps and usually means reading a lot of text to learn something a graph would
have shown in seconds.

**Correlate on `trace_id`.** It is the join key between the three, which is why the logging
conventions require it on every line. Without it you are correlating on timestamps, and
timestamps across services are close enough to mislead and not close enough to trust.

**Absence of evidence is common here.** Sampling drops traces, log retention expires, and a
metric may never have existed. "I could not find it" is not "it did not happen" - say which one
you mean when you report it, because the difference changes what the next person does.

**Beware of confirming a theory.** Every one of these tools will show you something interesting
if you look long enough. Decide what result would *disprove* your theory before you query, and
you will spend far less time on wrong ones.

---

## Our tools

<!-- FILL: map each class to what you actually use, with a link and one line on how to get
     access. Access matters more than it looks: a tool the new on-call cannot log into at 3am
     is a tool you do not have.

| Class | Tool | Where | Access | Retention |
|---|---|---|---|---|
| Metrics | | | | |
| Logs | | | | |
| Traces | | | | |
| Paging | | | | |
| Incidents | | | | |
| Deploy history | | | | |
| Flags | | | | |
| Status pages (third parties) | | | | |
-->

## Common queries

<!-- FILL: the five or six queries you actually run every time, written out in full and ready
     to paste. Error rate by service, latency percentiles for an endpoint, logs for a trace id,
     recent deploys, queue depth over time. Nobody composes a query language correctly while
     tired, and the ones you always need are few. -->

## Gaps

<!-- FILL: what you cannot see. Honestly. Every system has blind spots, and knowing "we have no
     tracing on the batch path" before an incident is worth more than discovering it during
     one. This section also doubles as the roadmap for what to instrument next. -->

## Programmatic access

If you have scripts that query these tools, they live in [`scripts/`](scripts/) and must follow
[`scripts/CONTRACT.md`](scripts/CONTRACT.md). The shape of what each tool returns is documented
in [`data-structure.md`](data-structure.md).
