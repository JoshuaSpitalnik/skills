# Repository and data-access tests

These prove that what you meant to store is what got stored, and that what you asked for is
what came back. That is not something a fake can tell you: a faked store returns whatever you
programmed it to return, so it confirms your assumptions rather than testing them. Constraints,
type coercion, null handling, index behavior, transaction semantics and migration correctness
only exist in a real engine.

So the rule for this category is the inverse of the others: **use the real database.**

## Scope

Tests in this category cover: queries and their filters, writes and their constraints,
transactions and rollback, concurrency behavior, and migrations in both directions.
They deliberately do not cover: business rules that happen to touch storage - those belong in
[`unit.md`](unit.md), where they run in milliseconds.

Location: <!-- FILL -->
Naming: <!-- FILL -->
How to run just these: <!-- FILL -->
How to get a database locally: <!-- FILL: the single command. If this is more than one step,
     these tests will be skipped locally and only fail in CI, which is the slowest possible
     feedback loop. -->

## Isolation

Isolation is the whole engineering problem of this category. Tests that leak state into each
other produce failures that depend on execution order, and order-dependent failures are close
to undebuggable.

<!-- FILL: your mechanism, e.g. each test runs in a transaction rolled back at teardown, or
     each test gets a freshly-created schema. State which, because the two have different
     limits - a rollback-based approach cannot test the code's own transaction handling, and a
     test that needs to commit must opt out. -->

Prefer rollback over cleanup code. Cleanup that runs after the assertions is cleanup that does
not run when a test fails - so one failure poisons every test after it, and the real cause is
buried in the noise.

## What to assert

- **Round-trip fidelity.** Write a record with every field populated - including the awkward
  ones: unicode, empty strings, nulls, maximum-length values, extremes of numeric range - read
  it back, and compare. Silent truncation and type coercion are found here or in production.
- **Filters actually filter.** Seed rows that must match *and* rows that must not, and assert
  both the inclusions and the exclusions. A query with a broken `WHERE` passes any test that
  only seeds matching rows.
- **Constraints are enforced.** Unique violations, foreign keys, not-null, and checks each
  raise the error your code expects to catch. Code that catches a constraint error it never
  provoked in a test is code that catches the wrong error.
- **Transactions roll back completely.** A failure mid-sequence leaves no partial state.
- **Ordering and pagination are stable.** A query without a deterministic sort returns rows in
  whatever order the engine likes, which changes as the table grows. Seed enough rows to expose
  it, and assert a total order.
- **Migrations apply and reverse** against a database holding representative data. A migration
  tested only against an empty schema tells you the syntax is valid, nothing more.

## What not to assert

- The generated SQL text. That is the implementation of the query, and it changes with every
  library upgrade while proving nothing about behavior.
- Query plans or execution time, in the functional suite. Performance regressions need their
  own harness with realistic volume; asserting timing here just produces a flaky test.
- Business rules layered above the query - push them down to unit tests.

## The trap in this category

**Seeding only rows that match.** The query returns them, the test is green, and the filter is
wrong. Every data-access test needs negative rows: the almost-matching record, the one belonging
to another tenant, the soft-deleted one. Tenant isolation in particular is worth an explicit
test per query - it is the failure mode with the worst consequences and the least visible
symptoms.

The second trap is a shared fixture that every test reads from. It starts as convenience and
becomes a coupling: nobody can change a row without breaking unrelated tests, so the fixture
grows instead, and eventually no test's preconditions are legible from the test itself. Prefer
per-test factories that state exactly what the test needs.

## Example

```
test "listing orders for a tenant excludes other tenants and soft-deleted rows":
    given  3 orders for tenant A, one of them soft-deleted
    and    2 orders for tenant B
    when   listing orders for tenant A
    then   exactly the 2 live tenant-A orders are returned
    and    they are ordered by created_at descending, then by id
```
