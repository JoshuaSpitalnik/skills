# Field reference

Every field on an alert or incident record, what it means, and - the column that matters -
**what it should change about your next action.** A field nobody knows how to act on is
decoration, and it costs attention at the worst possible moment. If you cannot fill the third
column for one of your fields, that is a finding: either the field needs a definition or it
needs deleting.

Fill these tables from your actual alerting and incident tooling. Names below are the common
ones; replace them with yours.

---

## Alert fields

These arrive with the page. They are what you have before you have looked at anything.

| Field | Means | So you should |
|---|---|---|
| `alert_name` / monitor name | The identity of the rule that fired | Look it up in [`../indexes/by-alert.md`](../indexes/by-alert.md) first. This is the highest-precision entry point you will get |
| `severity` | <!-- FILL: your levels and their definitions --> | <!-- FILL: what each obliges - who is notified, what response time, whether a status page updates --> |
| `service` | The system the rule is attached to | Open [`../../systems/docs/`](../../systems/docs/) for it - but see the warning below: this is the service that *noticed*, not necessarily the one that is broken |
| `environment` | <!-- FILL --> | <!-- FILL: your non-prod escalation policy - if it is "none", say so, because otherwise people guess --> |
| `fired_at` | When the condition was first met | Compare against deploy and flag history. The gap between `fired_at` and when the condition actually started is the evaluation window - see below |
| Evaluation window / `for` duration | How long the condition held before firing | Subtract it from `fired_at` to get the real start. Missing this makes you correlate against the wrong deploy |
| `threshold` and observed value | What tripped, and by how much | A value barely over threshold behaves differently from one ten times over. Barely-over during a known busy period is often a threshold problem, not an incident |
| Labels / tags | <!-- FILL: your dimensions - region, tenant, instance, endpoint, version --> | These are your scope answer. A label present on the alert but absent from others is the strongest scoping clue available |
| `runbook_url` | The written procedure | Follow it. If it is missing or stale, fix that as part of the incident - the next person is paged at 3am too |
| <!-- FILL --> | <!-- FILL --> | <!-- FILL --> |

**The `service` field names the service that noticed.** Alerts are attached to whatever observed
the symptom, which is usually downstream of the fault. Treat it as the starting point of a walk,
not the answer - [`nested.md`](nested.md) covers walking it backwards.

---

## Incident record fields

These accumulate during and after the incident. Their value is almost entirely in what they let
you find and compare later.

| Field | Means | So you should |
|---|---|---|
| `severity` | <!-- FILL: the same scale as alerts, or a separate one? Say which - the mismatch is a common source of confusion --> | <!-- FILL --> |
| `status` | <!-- FILL: your lifecycle states, e.g. investigating / identified / monitoring / resolved --> | <!-- FILL: what each transition obliges - who gets told, whether the status page changes --> |
| `impact` | Who is affected and what they cannot do | Write it in the user's terms. Everyone who joins later reads this field first, and a technical description makes each of them ask the same question |
| `started_at` | When impact actually began | Not when the alert fired, and not when you noticed. Correct it once you know - all duration metrics derive from it |
| `detected_at` | When a human or monitor knew | The gap to `started_at` is your detection time, which is the most improvable number in incident response and the one nobody records |
| `mitigated_at` | When impact stopped | Distinct from resolved. Confusing them makes every duration statistic meaningless |
| `resolved_at` | When the cause was actually fixed | May be days after mitigation, and that is fine as long as it is honest |
| Commander / lead | Who is coordinating | On anything above the lowest severity, one named person. Without it, three people run three investigations and none of them talks to the others |
| Affected services | Everything involved, not just the alerting one | Cross-check against [`../../systems/docs/internal-dependencies.md`](../../systems/docs/internal-dependencies.md); this list is what later reveals a shared dependency behind repeated incidents |
| Contributing changes | Deploys, flags, migrations implicated | Fill it even when uncertain, and say it is uncertain |
| Ruled out | What was investigated and eliminated | The field most often left empty and most useful to the next person. Fill it as you go, not at the end - by the end you will have forgotten most of it |
| Follow-up actions | What must change so this does not recur | Each needs an owner and a date, or it is a wish |
| <!-- FILL --> | <!-- FILL --> | <!-- FILL --> |

---

## Severity

Severity is the field that decides who gets woken up, so getting it wrong is expensive in both
directions - too high burns people, too low means nobody comes.

<!-- FILL: your levels. For each, define it by observable impact rather than by adjectives -
     "some users affected" is unusable at 3am, "checkout fails for more than 5% of attempts"
     is not. State for each: who is notified, expected response time, whether a status page
     updates, and whether a postmortem is required.

| Level | Definition (observable) | Notified | Response | Status page | Postmortem |
|---|---|---|---|---|---|
|  |  |  |  |  |  |
-->

**Raising and lowering severity mid-incident is normal.** Say it out loud when you do, and note
the time - a severity that silently changes makes the timeline unreadable, and people who joined
under the old severity keep acting on it.
