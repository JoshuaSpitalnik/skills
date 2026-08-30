# Cascading failures

The alert fires on the service that *noticed*, which is generally not the service that is
broken. A payment timeout pages checkout; a saturated database pages every service that reads
from it. Investigating the alerting service directly is the single most common way to lose the
first twenty minutes of an incident.

This guide is about walking from the symptom to the fault, and about telling apart the three
shapes those situations take.

---

## Three shapes, and how to tell them apart

**A chain.** A fails because B fails because C fails. One fault, propagating along dependency
edges. Alerts fire in sequence, innermost first if you look closely at the timestamps - though
detection delays often scramble the apparent order.

**A common cause.** A, B and C all fail at once because they share something - a database, an
auth provider, a network path, a deploy pipeline, a certificate. No dependency edge connects
them to each other. Alerts fire nearly simultaneously across services that have nothing to do
with one another.

**A coincidence.** Two unrelated problems at the same time. Rare, but it happens, and it is the
one that wastes the most time - because you build a theory connecting them and every new fact
has to be bent to fit.

The distinguishing evidence:

| Evidence | Chain | Common cause | Coincidence |
|---|---|---|---|
| Alert timing | Staggered, innermost first | Near-simultaneous | Unrelated times |
| Dependency path between the affected services | Exists | None | None |
| Shared infrastructure | Not necessarily | Yes - always find it | No |
| Fixing the innermost | Fixes everything | Fixes nothing | Fixes one |

That last row is the decisive test and usually the cheapest one available.

---

## Walking backwards

1. **Start at the alerting service.** Note that this is where the symptom was *observed*.

2. **Open its upstream list** in
   [`../../systems/docs/`](../../systems/docs/) - the "what it depends on" table. Check the
   `critical` and `degraded` edges before anything else; an `optional` dependency rarely
   produces a page.

3. **For each upstream, ask: is it healthy right now?** Not "did it alert" - a dependency can be
   degraded below its own alerting threshold while comfortably breaking its consumer. Its
   consumers have tighter tolerances than its own monitors do.

4. **Follow the first unhealthy edge and repeat.** Stop when you reach something whose upstreams
   are all healthy. That is your fault, or as close as the map can take you.

5. **If every upstream is healthy, the fault is here** - in this service, its data store, its
   configuration, its host, or its recent deploy. That is a genuine finding, not a dead end: it
   is what the walk was for.

<!-- FILL: your fastest way to check a service's health from outside it - one dashboard, one
     query, one command. This check runs several times per walk, so the difference between
     30 seconds and 3 minutes compounds. -->

---

## When several alerts fire together

Look for the **common dependency before opening several investigations.** Simultaneous
unrelated failures are rare; a shared component is the usual explanation.

- Cross-reference the affected services in
  [`../../systems/docs/internal-dependencies.md`](../../systems/docs/internal-dependencies.md),
  particularly its shared-infrastructure section. Shared components are easy to overlook
  precisely because no team owns them as a "service".
- Anything that touches everything is a candidate even when it has no alert of its own: DNS,
  auth, the event bus, the container platform, the deploy pipeline, an expiring root
  certificate.
- <!-- FILL: your shared components and how to check each is healthy. This list is the highest
     value thing on this page - fill it before anything else here. -->

**One commander, one investigation.** Several people independently investigating branches of
one cascade will each find a real-looking local problem, and the group will chase three
symptoms while the cause sits unexamined.

---

## Cascade amplifiers

Some failures are worse than their trigger because the system amplifies them. Recognizing the
amplifier matters, because it changes the fix: stopping the amplification often restores service
before the original fault is fixed at all.

**Retry storms.** A slow dependency causes retries, which multiply load, which makes it slower.
Load stays high even after the original trigger passes, so the system does not recover on its
own. Symptom: request volume to the struggling service far above normal, and recovery that never
comes. Fix: shed load or disable retries first, then address the trigger.

**Connection pool exhaustion.** Slow responses hold connections; the pool empties; requests that
have nothing to do with the slow path start queueing. Symptom: everything through one service is
slow, including endpoints that touch nothing broken.

**Queue backpressure.** A stalled consumer lets a queue grow; producers block or drop. Symptom:
the *producer* is what alerts, and the consumer looks idle rather than failing.

**Cache stampede.** A cache empties or expires en masse and every request goes to the origin at
once. Symptom: origin load spikes far above what user traffic would explain.

**Timeout mismatch.** A caller's timeout is shorter than its callee's, so the caller gives up and
retries work that is still in progress. Symptom: duplicated work, and a downstream far busier
than the request rate suggests.

<!-- FILL: the amplifiers you have actually experienced, with the service and the fix. Yours
     will recur - the underlying settings rarely change after an incident. -->

---

## Recording the walk

Write down the path you took and, for each service, whether you found it healthy. This is the
most reusable artifact an investigation produces: it prevents the next person re-walking the
same edges, and after a few incidents the pattern of which edges keep appearing tells you where
the fragility actually lives.

If the walk revealed a dependency the map did not have, add it to both service docs before you
close. A missing edge will cost the next person the same twenty minutes it cost you.
