# Scheduled job tests

> **Illustrative example.** `orderflow` is a fictional service used to show the intended shape
> and density of a pattern file. Nothing here describes real infrastructure. Delete this file
> once your own pattern files make it redundant.

Scheduled jobs fail differently from request-handling code: nobody is waiting on the result, so
a failure is silent until its absence causes a second failure somewhere else. These tests exist
to prove a job is safe to run at an unexpected time, twice, or not at all - because in
production, all three eventually happen.

## Scope

Tests in this category cover: the job's decision logic, its behavior on a partially-completed
previous run, and its behavior when its window is missed.
They deliberately do not cover: whether the scheduler actually fires - that is infrastructure,
and asserting it here produces a test that passes while the job never runs.

Location: `tests/jobs/<job_name>_test.*`
Naming: `<job>_<condition>_<expected outcome>`, e.g. `nightly_settlement_skips_already_settled_orders`
How to run just these: `test --tag=jobs`

## What is real and what is faked

| Dependency | Real or faked | Why |
|---|---|---|
| The job's own logic | Real | It is the unit under test |
| The clock | Faked, injected | The whole category is about behavior at specific times; a real clock makes the interesting cases untestable and the boring ones flaky |
| The database | Real, in a per-test transaction | Jobs are mostly queries, and a faked store proves the fake works, not the query |
| Outbound notifications | Faked at the client interface | We own the interface; the vendor's delivery is not our behavior to assert |
| The scheduler | Absent | The job is invoked directly - see scope |

## Setup

Each test constructs the job with an explicit `now`, seeds the database inside a transaction
rolled back at teardown, and invokes the job's entry point directly. Isolation comes from the
transaction, not from cleanup code - cleanup that runs after a failing test is cleanup that
sometimes does not run.

## What to assert

That the correct records were selected and changed; that already-processed records were left
alone; that a second invocation with the same clock is a no-op; that a missed window is
recovered from rather than skipped.

## What not to assert

The number of queries issued, the order of processing within a batch, or the exact log lines.
All three change under harmless refactors, and none of them is behavior anyone depends on.

## The trap in this category

Writing every test at the job's "happy" scheduled time. The failures that actually page someone
come from a job running twice after a retry, or running at 00:59 on the night the clocks change.
Because the clock is injected, those cases cost one line each - the trap is not that they are
hard, it is that nobody thinks of them.

## Example

```
test "nightly settlement skips already-settled orders":
    given  now = 2026-03-14T02:00:00Z
    and    an order settled at 2026-03-13T02:00:00Z
    and    an order captured but not settled
    when   the settlement job runs
    then   only the unsettled order is settled
    and    running the job again changes nothing
```
