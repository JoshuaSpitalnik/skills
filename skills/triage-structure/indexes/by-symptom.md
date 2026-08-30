# Index: symptom → ranked suspects

For when you have a described behavior rather than an alert - a report from a colleague, a
customer complaint, or your own observation. Less precise than
[`by-alert.md`](by-alert.md), so the value here is in the **ranking**: which suspect to check
first, and how to eliminate it quickly.

Rows say where to look. They do not hold service facts - those live in
[`../../systems/docs/`](../../systems/docs/).

---

| Symptom | Most likely | How to confirm or eliminate | Then check | Then | Notes |
|---|---|---|---|---|---|
| <!-- FILL: described the way someone would actually report it --> | <!-- FILL --> | <!-- FILL: the cheapest decisive check --> | <!-- FILL --> | <!-- FILL --> | <!-- FILL --> |
|  |  |  |  |  |  |

---

## Filling this in

**Phrase the symptom the way it will be reported,** not the way you would describe it. People
say "checkout is stuck", not "elevated p99 on the order submission path". If the phrasing here
does not match the phrasing in your incident channel, the row will not be found. Add the
alternate phrasings people actually use as extra rows pointing at the same place - duplication
of *pointers* is fine; duplication of *facts* is not.

**Rank by expected time saved, not by probability alone.** A cheap check that eliminates a
common cause beats an expensive check that eliminates a slightly more common one. The first
column should be whatever costs the least to rule out.

**"How to confirm or eliminate" must be a specific, fast action.** A named dashboard, a query, a
command. This column is the whole point of the file - a list of suspects with no way to test
them is just a list of things to worry about.

**Elimination is as valuable as confirmation.** A row that lets someone rule out a subsystem in
thirty seconds has paid for itself. Write the checks so the negative result is as clear as the
positive one.

## Symptom shapes worth having rows for

Most ecosystems need rows for roughly these, however they manifest for you. Use them as a
starting list rather than a schema:

- **Slow, but working** - latency without errors. Usually a dependency, a lock, or a saturated
  pool rather than a bug.
- **Failing for some users, fine for others** - the split is the clue: by tenant, region,
  client version, or a flag. Find the axis before finding the cause.
- **Failing intermittently** - one bad instance, a partial deploy, or an intermittent dependency.
- **Nothing is happening** - a stalled consumer or a job that did not run. Produces no errors at
  all, so it is usually reported by a person; see
  [`../guides/nested.md`](../guides/nested.md) on silent failures.
- **Backlog growing** - a consumer slower than its producer, or stopped entirely. Check
  consumption rate before assuming a failure; a working-but-slow consumer looks identical from
  the queue's side.
- **Stale or wrong data** - a cache not invalidating, replication lag, or an event not published.
- **Worked yesterday, broken today, nothing deployed** - look for expiry (certificates,
  credentials, tokens), scheduled changes, data volume crossing a threshold, or someone else's
  change.

<!-- FILL: turn each of these into a real row with your services in it, and delete the ones
     that do not apply to you. -->

## Maintenance

When an investigation starts from a symptom, the path you took is a row - write it while it is
fresh. Rows added this way are better than rows written speculatively, because they encode the
order someone actually found useful rather than the order that seemed sensible in the abstract.
