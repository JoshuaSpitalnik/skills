# Index: alert name → where to look

The highest-precision entry point available: you have the exact identity of the rule that
fired. One row per alert, keyed on the alert or monitor name **exactly** as it appears on the
page - if the string here differs from the string in the alerting system, nobody finds the row.

Rows stay thin. They say where to go, never what the fix is: the fix lives in the runbook, and
the service's facts live in [`../../systems/docs/`](../../systems/docs/). A row that repeats a
service's dependencies is a row that will contradict them within a quarter.

Sorted alphabetically by alert name so it can be scanned under pressure.

---

| Alert name | Owning system | What it actually means | First checks | Runbook | Known false positives |
|---|---|---|---|---|---|
| <!-- FILL: exact string from the page --> | <!-- FILL: link to ../../systems/docs/<service>.md --> | <!-- FILL: what condition tripped, in plain terms - not a restatement of the name --> | <!-- FILL: two or three, in order --> | <!-- FILL --> | <!-- FILL: or "none known" --> |
|  |  |  |  |  |  |

---

## Filling this in

**"What it actually means" is the column that earns its keep.** `HighLatency` restated as "high
latency" helps nobody. "p99 on the read path exceeded 2s for 5 minutes - note this is the read
path only, writes have a separate alert" ends an entire round of confusion.

**"First checks" are ordered and specific.** Two or three, the cheapest and most decisive first.
"Investigate the service" is not a check. "Confirm on the `orderflow-overview` dashboard that
the latency is on the read path, then check whether `payments` p99 rose in the same window" is.

**"Known false positives" is written after an alert fools someone,** which means it starts empty
and fills up through use. It is the column that most repays the discipline, because a
false-positive check costs seconds and a false-positive investigation costs an hour. Record what
the false version looks like, not just that one exists: "fires during the 02:00 batch window;
if `started_at` is between 02:00 and 02:30 UTC and only the replica is affected, it is the batch
job."

**Every alert defined on a service should appear here,** and every service doc's alert list
should match. When you add an alert, add the row. An alerting rule with no row pages someone
who then has nowhere to start - which is precisely the situation this file exists to prevent.

## Maintenance

- After an incident: refine the row you used. If it was wrong or thin, that is the highest-value
  edit available to you and you will never have better context for making it.
- After a false positive: add or extend the last column.
- After deleting an alert: delete the row. A row for an alert that no longer exists sends
  someone looking for a monitor they will not find.
- Periodically: reconcile against the alerting system. Alerts drift, get renamed, and get
  silently disabled; the rename is the dangerous one, because the row still exists but nobody
  matching on the new name will find it.
