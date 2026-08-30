# Anti-patterns

Things that look reasonable in the moment and cost someone a day later. Each row states the
cost, because a rule without a stated cost gets treated as taste and argued with; a rule with
one gets followed.

Read this before a review, and whenever a solution feels clever. Cleverness is usually a
correct answer to the wrong question.

## Universal

These hold in any ecosystem. Keep them; add yours below.

| Don't | Why it bites | Do instead |
|---|---|---|
| Catch an exception and continue with no log or comment | The failure becomes invisible; the bug report is "sometimes the total is wrong" with no evidence anywhere | Handle it, or let it propagate. If genuinely ignorable, log at debug with the reason |
| Build log messages by string interpolation | Every occurrence is a unique string, so it cannot be counted, alerted on, or indexed - the `by-error` index has nothing to key on | Stable `event` slug plus structured fields ([conventions.md § Logging](conventions.md#logging)) |
| Retry at several layers of the stack | Three layers of 3 retries is 27 requests; a brief downstream blip becomes a self-inflicted outage | Retry at exactly one boundary, the one that owns the dependency |
| Add a boolean parameter to change what a function does | Call sites read `process(order, true, false)` and every reader has to open the definition | Two named functions, or an explicit option type |
| Reach across a layer boundary "just this once" | The exception becomes the precedent; six months later the layering exists only in the docs | Route through the interface, or change the layering deliberately |
| Widen a type or interface to accommodate one caller | Every other implementer now handles a case that cannot occur for them | A separate interface, or push the special case to where it is actually special |
| Copy a fact into a second file | The copies diverge, and nobody can tell which is current - especially costly in triage indexes | Keep one source of truth and link |
| Leave commented-out code | Readers cannot tell if it is a hint, a rollback plan, or an oversight, so nobody dares delete it | Delete it. Git remembers |
| Fix a flaky test by adding a sleep | The race is still there, now slower and less reproducible | Wait on the actual condition, or fix the race |
| Name a module `utils`, `common`, `helpers`, or `misc` | Nothing is obviously *not* a utility, so it grows forever and becomes an import cycle | A named module per concept, even a small one |
| Rename a public field alongside a behavior change | When it breaks, nobody can tell which half did it | Two changes, two commits |
| Gate a code path on the environment name | Staging stops being a test of production; the untested path is the one that runs for customers | A named config or feature flag ([settings.md](settings.md)) |

## Ecosystem-specific

<!-- FILL: the mistakes that actually recur in your code. The best source is your own review
     comments - if you have written the same comment three times, it belongs here. The second
     best source is post-incident reviews: every incident with a "we should have..." produces
     exactly one row.

| Don't | Why it bites | Do instead |
|---|---|---|
|  |  |  |
-->

## Retired

<!-- FILL: rules you have deliberately dropped, and why. Keeping the graveyard visible stops
     someone re-litigating a settled decision, and stops a rule surviving past the constraint
     that justified it. Delete this section if empty. -->
