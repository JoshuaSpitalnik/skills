---
name: triage-structure
description: On-call incident and alert investigation for this ecosystem - turn a page, alert name, symptom, or error signature into the right system, the right first checks, and the right runbook. Use this skill whenever the user is on call, got paged, mentions an alert or monitor firing, an incident, an outage, a degradation, prod being slow or broken, a spike in errors or latency, a stuck queue, or pastes a stack trace, log line, or status code and asks what is going on - even when they never use the words incident or triage.
---

# On-call triage

The job in the first minutes is not to find the root cause. It is to work out **what is
affected, whether it is getting worse, and what stops it** - then diagnose. Investigating a
still-spreading failure is how a fifteen-minute incident becomes an hour.

Everything here is a router. Go to the one entry point that matches what you actually have in
hand, rather than reading the whole tree.

## Start here

| You have | Go to |
|---|---|
| An alert or monitor name from a page | [`indexes/by-alert.md`](indexes/by-alert.md) |
| A described behavior - "checkout is slow", "the queue is backing up" | [`indexes/by-symptom.md`](indexes/by-symptom.md) |
| A log line, exception, or status code | [`indexes/by-error.md`](indexes/by-error.md) |
| A named service, and you need to know what it does or who owns it | [`../systems/docs/`](../systems/docs/) |
| Nothing solid yet - "something is wrong with prod" | [`guides/quick.md`](guides/quick.md) |
| A suspicion this alert is downstream of the real problem | [`guides/nested.md`](guides/nested.md) |
| An incident record whose fields you need to interpret | [`guides/field-reference.md`](guides/field-reference.md) |
| A question about which observability tool to ask | [`tools/README.md`](tools/README.md) |

An index tells you **where to look**. It never tells you the fix - that lives in the runbook it
links to, and in the service's own doc under
[`../systems/docs/`](../systems/docs/). Keeping the split means a service's facts have one home
and cannot rot into two contradictory versions.

## How to run an investigation

**Confirm it is real before you act on it.** Check that the symptom is visible from a second,
independent source - the alert plus a dashboard, or the error rate plus an actual failing
request. Monitors break, and a plausible false positive that gets "fixed" wastes the window
during which the real problem is still spreading. The `by-alert` index records known false
positives per alert; check that column first.

**Establish scope before cause.** Who is affected, how many, since when, and is the number
growing? These four answers determine severity, whether to escalate, and how much time you
have. They are also what anyone joining the incident needs first, so having them written down
saves you repeating yourself.

**Ask what changed.** The great majority of incidents start with a change: a deploy, a config
or flag flip, a migration, a traffic shift, an expiring credential, or a change in someone
else's system. Check deploy and flag history early - it is cheap and it resolves a large share
of investigations outright.

**Stabilize before diagnosing.** If a rollback, a flag flip, or pausing an entry point stops
the harm, do it now. You can investigate a stable system for as long as you need to; you cannot
investigate a spreading one. Note what you changed and when, so the timeline stays honest.

**Write down what you ruled out, not just what you suspect.** Most of an investigation's value
to the next person is the eliminations. Someone joining an hour in needs to know which paths are
already dead more than they need your current hunch - and without that list they will re-check
the same things you did.

**Say when you are stuck.** Escalation is a normal move, not a failure. The threshold is time
and impact, not certainty:
<!-- FILL: your escalation rule and contacts, e.g. "SEV1: page the service owner immediately.
     SEV2: escalate if not understood within 30 minutes. Contacts per service are in that
     service's doc." -->

## Common shapes

**The alerting system is not the broken system.** A page fires on the service that noticed,
which is usually downstream of the fault. Before deep-diving the alerting service, check its
upstream dependencies - [`guides/nested.md`](guides/nested.md) covers walking the edge
backwards.

**Several alerts at once usually means one cause.** Resist opening several investigations.
Look for the common dependency in
[`../systems/docs/internal-dependencies.md`](../systems/docs/internal-dependencies.md);
simultaneous unrelated failures are rare, and shared infrastructure is the usual answer.

**Silence can be the symptom.** An asynchronous consumer that stops doing work produces no
errors at all - just an absence. If someone reports that something did not happen, look for a
stalled consumer or job before looking for a failure.

## Keeping this useful

Every incident should leave a trace here: a new row in the index you wished had existed, a
false-positive note on an alert that fooled you, or a failure mode added to the service's doc.
An index nobody adds to after an incident goes stale in one quarter, and an index that is wrong
at 3am costs more than one that is empty - because someone believes it.

Never copy a service fact into an index. Link to
[`../systems/docs/`](../systems/docs/) instead.
