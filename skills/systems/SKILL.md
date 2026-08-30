---
name: systems
description: Map of the internal systems in this ecosystem and how they depend on each other - what each service owns, what it calls, what calls it, where its data lives, and how it fails. Use this skill whenever a question spans more than one service, before tracing a request path across services, when assessing the blast radius of a change or a deploy, when answering "what breaks if X goes down" or "who owns this behavior", and whenever an on-call investigation needs to know what sits upstream or downstream of a failing component.
---

# The system map

This is the single source of truth for what each internal system is, what it depends on, and
what depends on it. Other skills link here rather than restating - a service's owner or
dependency list written down in two places will disagree within a quarter, and during an
incident nobody can tell which copy is current.

## Which file to read

| You need | Read |
|---|---|
| Everything about one service | [`docs/<service>.md`](docs/) |
| Who calls whom, and how badly it hurts | [`docs/internal-dependencies.md`](docs/internal-dependencies.md) |
| Blast radius of a change or an outage | [`docs/internal-dependencies.md`](docs/internal-dependencies.md), then each affected service's doc |
| To add a service | Copy [`docs/_TEMPLATE.md`](docs/_TEMPLATE.md); [`docs/_EXAMPLE.md`](docs/_EXAMPLE.md) shows one filled in |

Read one service doc, not all of them. The tree is split per service so that a question about
one does not pull the whole estate into context.

## How to use the map

**Start from the question, not from the top of the graph.** "What breaks if the payments
service is down?" is answered by that service's *downstream consumers* list, not by reading the
whole dependency table.

**Follow edges in the direction of the question.** Upstream (what this calls) answers "why is
this failing". Downstream (what calls this) answers "who will notice". They are different lists
and confusing them is the most common mistake made under time pressure.

**Criticality is about the consumer, not the provider.** A dependency is critical if the caller
cannot serve its purpose without it. The same provider can be critical to one consumer and
optional to another, which is why criticality is recorded per edge.

**Trust the map only as far as its dates.** Each service doc carries a last-reviewed date. A doc
older than your last big migration is a hypothesis - verify before acting on it, and update it
while you are there.

## When this map is wrong

Fix it in the same session you noticed. A stale system map is worse than no map, because it is
consulted with confidence at exactly the moment when being wrong is expensive. If you cannot
verify the correct answer immediately, mark the entry as uncertain rather than leaving a
confident falsehood in place.

## Relationship to the other skills

- **On-call and incidents** are `triage-structure`. Its indexes intentionally hold no service
  facts - they link here. If you are about to write a service's dependencies into a triage
  index, write them here and link instead.
- **Code conventions** are `conventions-writing`. Note that the `service` field required in
  every log line must match the service name used here, exactly - that string is what joins
  logs to this map during an investigation.

<!-- FILL: your service inventory, one line each, as a jump table. Keep it to a line per
     service; the detail lives in the per-service doc.

| Service | Owns | Doc |
|---|---|---|
|  |  | [`docs/<name>.md`](docs/<name>.md) |
-->
