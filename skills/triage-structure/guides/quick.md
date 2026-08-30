# The first five minutes

For when you have just been paged, or someone says "prod is weird" and you have nothing solid
yet. Work down the list. Do not skip to diagnosis - the earlier steps are what tell you how
much time you have.

Keep a running note as you go. It becomes the timeline, and you will not remember the order of
events afterwards.

---

## 1. Is it real?

Confirm the symptom from a source independent of whatever alerted you. An alert plus a
dashboard, or an error rate plus one actually-failing request.

- Check the alert's row in [`../indexes/by-alert.md`](../indexes/by-alert.md) - the
  false-positive column records the ones that have fooled people before.
- <!-- FILL: your one-line "is the system alive" check - a health endpoint, a status page,
     a synthetic transaction. Name the exact thing to look at. -->

**If it is not real:** say so in the incident channel, note it against the alert's row so the
next person does not repeat the check, and fix or retune the monitor. A false positive left
unrecorded fires again next month.

## 2. Who is affected, and how many?

- Is it all users, one tenant, one region, one endpoint, one internal consumer?
- What can they not do? Answer in the user's terms - "cannot check out" beats "500s on
  `POST /orders`" for everyone who joins later.
- Is the number growing, steady, or shrinking?

<!-- FILL: where you see user-facing impact quickly - the dashboard, the query, the graph.
     One link. -->

**Growing changes everything.** A steady failure can be diagnosed carefully. A growing one gets
stabilized first (step 4).

## 3. What changed?

Most incidents have a change behind them, and this check is fast enough to do before anything
clever.

- Deploys in the last few hours - any service, not just the alerting one
- Feature flag or config changes
- Migrations
- Traffic shape - a spike, a new client, a retry storm
- Expiring things - certificates, credentials, tokens
- Someone else's change: a vendor, a shared platform, a cloud provider status page

<!-- FILL: where to see recent deploys and flag flips across all services, in one place if you
     have it. If you do not have one place, list the two or three you do have - the point is
     that this check must take under a minute or it will be skipped. -->

**If a recent change lines up in time, that is your prime suspect.** Correlation in time is not
proof, but it is the highest-yield hypothesis available and it is usually testable by reverting.

## 4. Can you stop the bleeding?

Before diagnosing, ask whether something makes the harm stop now. Reversible actions are
strongly preferred - you can undo them if you were wrong.

| Option | When it fits | Note |
|---|---|---|
| Roll back a deploy | A change lines up in time | Rollback procedure and duration are in the service's doc under [`../../systems/docs/`](../../systems/docs/). Check whether the deploy included a migration |
| Flip a feature flag off | The failure is behind a flag | Fastest lever available, and usually the safest |
| Pause an entry point | A consumer or job is causing the damage | Entry points and their pause-safety are in the service doc |
| Shed or rate-limit load | Overload or a retry storm | Buys time; is not a fix |
| Fail over | A single component is unhealthy | Only where the runbook says so |

<!-- FILL: which of these you actually have, and the exact command for each. Under pressure
     nobody reconstructs a procedure from principles. -->

**Record what you changed and at what time.** An unrecorded mitigation makes the rest of the
timeline unreadable - later effects get attributed to the wrong cause.

## 5. Set severity and decide about escalation

<!-- FILL: your severity definitions and what each obliges you to do - who is notified, whether
     a status page updates, whether a comms person is pulled in. See
     ./field-reference.md for the field itself. -->

Escalate on **time and impact, not on certainty**. If you do not understand it within your
threshold, bring in the service owner - handing over a well-documented unknown is a good
outcome, not an admission of anything.

## 6. Now diagnose

You have a stable system, a scope, and a suspect. Pick the entry point that matches what you
have:

- An alert name → [`../indexes/by-alert.md`](../indexes/by-alert.md)
- A symptom → [`../indexes/by-symptom.md`](../indexes/by-symptom.md)
- An error signature → [`../indexes/by-error.md`](../indexes/by-error.md)
- A suspicion the alerting service is not the broken one → [`nested.md`](nested.md)
- Not sure which tool answers your question → [`../tools/README.md`](../tools/README.md)

As you go, write down what you **ruled out**. It is the part of the investigation with the
longest useful life.

## 7. Before you close

- [ ] Confirm recovery from the same independent source you used in step 1
- [ ] Revert any temporary mitigation, or write down that it is still in place and who owns
      removing it - temporary mitigations that nobody tracked are a recurring source of the
      next incident
- [ ] Add what you learned: a row in the index you wished had existed, a false-positive note, a
      failure mode in the service's doc under [`../../systems/docs/`](../../systems/docs/)
- [ ] <!-- FILL: your incident record / postmortem trigger and threshold -->
