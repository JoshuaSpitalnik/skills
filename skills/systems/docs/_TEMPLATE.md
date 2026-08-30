# <!-- FILL: service name, exactly as it appears in log lines and alert labels -->

> Last reviewed: <!-- FILL: YYYY-MM-DD --> by <!-- FILL -->
> Status: <!-- FILL: active / deprecated / being replaced by X -->

## What it owns

<!-- FILL: one paragraph. The test of a good answer: someone reading it can decide whether a
     given behavior belongs to this service or another one. If the paragraph is a list of
     technologies rather than responsibilities, it will not pass that test. -->

**It is not responsible for:** <!-- FILL: the things people reasonably assume it does, and
     which service actually does them. This line saves more time during an incident than
     anything else on the page. -->

## Ownership

| | |
|---|---|
| Team | <!-- FILL --> |
| On-call rotation | <!-- FILL --> |
| Escalation after hours | <!-- FILL: who, and at what severity - "escalate if needed" is not actionable at 3am --> |
| Source | <!-- FILL: repo and path --> |

## Upstream - what it depends on

| Depends on | Via | Criticality | If it is down |
|---|---|---|---|
| <!-- FILL --> | <!-- FILL: sync HTTP / async event / shared database --> | <!-- FILL: critical / degraded / optional --> | <!-- FILL: the observable consequence, not "errors" --> |

<!-- Criticality here means: can this service still do its job without that dependency?
     critical = no. degraded = partially, and say which part. optional = yes, unnoticed. -->

## Downstream - what depends on it

| Consumer | Via | Criticality to them | What they see if this is down |
|---|---|---|---|
| <!-- FILL --> | <!-- FILL --> | <!-- FILL --> | <!-- FILL --> |

<!-- This is the blast-radius list, and it is the one people forget to update. When adding a
     consumer, add the row here as well as in that consumer's upstream table. -->

## Data

| Store | Contains | Retention | Notes |
|---|---|---|---|
| <!-- FILL --> | <!-- FILL --> | <!-- FILL --> | <!-- FILL: shared with anyone? single point of failure? --> |

## Entry points

<!-- FILL: how work arrives - endpoints, consumed queues or topics, scheduled jobs, manual
     operations. For each, note whether it can be safely paused, because "stop the bleeding"
     during an incident usually means pausing one of these. -->

## Configuration and deploys

| | |
|---|---|
| How it deploys | <!-- FILL --> |
| Typical deploy duration | <!-- FILL --> |
| How to roll back | <!-- FILL: the exact command or procedure. Under pressure, nobody reconstructs this --> |
| Feature flags that change its behavior | <!-- FILL --> |

## Known failure modes

| Failure | Looks like | Usual cause | First action |
|---|---|---|---|
| <!-- FILL --> | <!-- FILL: the observable symptom --> | <!-- FILL --> | <!-- FILL --> |

<!-- Every incident should add or refine a row here. If a service has had three incidents and
     no rows, the knowledge is living in one person's head. -->

## Capacity and limits

<!-- FILL: known ceilings - rate limits it enforces, rate limits it is subject to, connection
     pool sizes, queue depth before back-pressure. Numbers, where you have them. An unstated
     limit is discovered by hitting it. -->

## Links

- Dashboards: <!-- FILL -->
- Alerts defined for it: <!-- FILL: and confirm each is listed in ../../triage-structure/indexes/by-alert.md -->
- Runbooks: <!-- FILL -->
- Design docs / ADRs: <!-- FILL -->
