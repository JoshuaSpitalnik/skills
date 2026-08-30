# Core conventions

Four sections, in the order you hit them while writing code: what to call it, where to put it,
what to do when it breaks, and what to record about it.

- [Naming](#naming)
- [Structure](#structure)
- [Errors](#errors)
- [Logging](#logging)

---

## Naming

Names are the cheapest documentation in the codebase and the most expensive to change later.
The goal is predictability: someone who knows the rule should be able to guess the name of a
thing they have never seen.

| Thing | Convention | Example |
|---|---|---|
| Directories | <!-- FILL: e.g. lowercase, hyphenated, plural for collections --> | <!-- FILL --> |
| Source files | <!-- FILL --> | <!-- FILL --> |
| Test files | <!-- FILL: and note whether they sit beside the source or in a mirror tree --> | <!-- FILL --> |
| Types / classes | <!-- FILL --> | <!-- FILL --> |
| Functions / methods | <!-- FILL: e.g. verb-first; `get` for cheap reads, `fetch` for I/O --> | <!-- FILL --> |
| Booleans | <!-- FILL: e.g. `is_`/`has_`/`should_` prefix, never negated --> | <!-- FILL --> |
| Constants | <!-- FILL --> | <!-- FILL --> |
| Config keys | <!-- FILL: see settings.md for the full rules --> | <!-- FILL --> |
| Branches | <!-- FILL: e.g. `<type>/<ticket>-<slug>` --> | <!-- FILL --> |
| Commits | <!-- FILL: e.g. conventional commits, imperative mood, ticket in footer --> | <!-- FILL --> |

**Rules that hold regardless of the table:**

- A name says what the thing *is or does*, not how it is implemented. `retry_queue` survives a
  rewrite; `redis_list` does not.
- Negated booleans compound badly. `is_disabled` becomes `if not is_disabled` at every call
  site, which readers misparse. Name the positive.
- Abbreviate only where the abbreviation is more common than the word in your domain (`id`,
  `url`, `http`). Everything else is spelled out - the keystrokes you save are paid back by
  every future grep that misses.
- The same concept keeps the same name across layers. If the database calls it `account` and
  the API calls it `customer`, every conversation about the system needs a translation step.

<!-- FILL: the abbreviations and domain terms specific to your ecosystem, and their one true
     spelling. This short glossary prevents more confusion than any other section here. -->

---

## Structure

Where a file goes is a claim about what depends on what. Layout that contradicts the real
dependency graph makes the graph invisible, and an invisible graph is one nobody can reason
about during an incident.

**Layers, outermost to innermost:**

<!-- FILL: name your layers and what belongs in each, e.g.
     1. entrypoints  - HTTP handlers, CLI commands, job triggers. No business logic.
     2. services     - use cases and orchestration. No I/O details.
     3. domain       - entities and rules. No imports from any outer layer.
     4. adapters     - database, queues, third-party clients. Implement interfaces owned inward.
-->

**Allowed import directions:**

<!-- FILL: state the rule and, critically, how it is enforced - a lint rule, a CI check, or
     honor system. An unenforced layering rule is a wish. -->

**Where a new file goes:**

| You are adding | It belongs in | Because |
|---|---|---|
| <!-- FILL: e.g. a new endpoint --> | <!-- FILL --> | <!-- FILL --> |
| <!-- FILL: e.g. a new third-party integration --> | <!-- FILL --> | <!-- FILL --> |
| <!-- FILL: e.g. a shared helper used by two services --> | <!-- FILL --> | <!-- FILL --> |

**On shared utility modules:** a `utils` or `common` directory is where code goes when nobody
decided what it is. It grows without bound because nothing is ever obviously *not* a utility.
Prefer a named module (`retry`, `pagination`, `money`) even when it holds one function - the
name forces the decision, and the decision is the useful part.

<!-- FILL: your rule for when a module earns its own package/service, and who decides. -->

---

## Errors

The question every error handler answers is: **who is going to act on this, and what do they
need in order to act?** An error that reaches a human without enough context to act is noise,
and noise trains people to ignore the channel it arrives on.

**Classify first.** Most codebases need roughly these categories - name yours and be consistent:

| Category | Meaning | Handling |
|---|---|---|
| Invalid input | The caller sent something we cannot accept | Reject at the boundary with a specific message naming the field. Do not log as an error - it is not our fault and it will drown real failures. |
| Not found | The thing asked for does not exist | Distinguish from "you may not see it" only where that distinction is not itself a leak. |
| Conflict | The request contradicts current state | Return what the current state is, so the caller can decide whether to retry. |
| Transient / dependency | A downstream failed and might succeed next time | Retry with backoff at the boundary that owns the dependency, then surface. Retrying at three layers turns one outage into a load test. |
| Bug | An invariant we believed was violated | Fail loudly, include enough state to reproduce, alert. |

<!-- FILL: adjust the table to your actual categories and add the type or code each maps to. -->

**Rules:**

- **Never swallow silently.** An empty catch block is a decision to make future debugging
  impossible. If an error is genuinely ignorable, log at debug level and write the reason - the
  reason is what a reader needs, not the suppression.
- **Wrap with context, do not replace.** Each layer adds what it knows (`"charging order
  a1b2"`) and preserves the cause. A re-raised error that discards the original stack costs a
  whole debugging session.
- **Do not log and re-raise.** The handler at the top logs once. Logging at every level
  produces five entries for one failure, and the on-call reads them as five failures.
- **Retry only idempotent operations,** and only at the boundary that knows the operation is
  idempotent. See [`api.md`](api.md) for idempotency keys.
- **Error messages name the thing and the constraint.** "Invalid value" is unactionable;
  "expected `currency` to be a 3-letter ISO code, got `dollars`" ends the investigation.

<!-- FILL: your error envelope for user-facing surfaces (link to api.md if it lives there),
     and which categories page a human vs. which only get recorded. -->

---

## Logging

Logs are the primary evidence available during an incident, and the fields defined here are
exactly what makes the `triage-structure` skill's
[`by-error` index](../../triage-structure/indexes/by-error.md) possible: an index keyed on error
signatures only works if the same failure produces the same, greppable signature every time.
That coupling is the reason both skills live in one repo. Log lines are for your future self at
3am, not for you right now.

**Levels - the test is "who acts, and when":**

| Level | Use when | Someone acts |
|---|---|---|
| `error` | An operation failed and a human needs to know | Yes, now or next business day |
| `warn` | Degraded but handled - a retry succeeded, a fallback engaged | Only if it becomes frequent |
| `info` | A significant state change worth reconstructing later | No, but it appears in every timeline |
| `debug` | Detail useful while investigating a specific problem | No, off in production |

If nobody would ever act on an `error`, it is a `warn`. If a `warn` fires continuously, it is
either an `info` or an unfixed bug - both mean the level is wrong.

**Every log line carries these fields:**

<!-- FILL: your required structured fields. A workable starting set:
     | Field | Meaning |
     |---|---|
     | `timestamp`     | UTC ISO-8601 |
     | `level`         | as above |
     | `service`       | must match the name in systems/docs/<service>.md, exactly |
     | `env`           | prod / staging / dev |
     | `trace_id`      | propagated across service boundaries, or a timeline cannot be built |
     | `event`         | a stable, greppable slug - see below |
     | `error_code`    | on failures; the key the by-error index is built on |
-->

**The `event` field is the one that matters.** Give each meaningful occurrence a stable slug
(`payment.capture.failed`) that never changes and never interpolates a variable. Variable data
goes in its own fields. This is what lets someone count occurrences, alert on a rate, and look
the failure up in an index. A message built by string interpolation is unsearchable and
un-aggregatable - every occurrence looks like a different failure.

**Never log:** credentials, tokens, API keys, full payment details, personal data beyond an
opaque identifier, or whole request bodies from an untrusted source. Logs are retained longer,
copied more widely, and access-controlled more loosely than any database you have.

<!-- FILL: your redaction mechanism and the exact list of fields it must scrub. -->

**On volume:** a log line inside a hot loop is a line nobody reads and a bill somebody pays.
Log the decision, not each iteration - the aggregate belongs in a metric.
