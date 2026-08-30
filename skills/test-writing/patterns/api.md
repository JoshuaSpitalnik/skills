# API and contract tests

These prove the promise you published is the promise you keep. Their audience is your callers,
who cannot see your internals and can only observe status codes, payloads, and headers - so
that is exactly what these tests observe too.

They are slower and fewer than unit tests. Do not use them to cover business rules that a unit
test could cover faster; use them to cover the boundary itself.

## Scope

Tests in this category cover: routing, request validation and rejection, response shape and
status codes, authentication and authorization, error envelopes, pagination, idempotent replay,
and backward compatibility.
They deliberately do not cover: the depth of the business logic behind the endpoint - one
representative success path is enough here; the branch matrix belongs in
[`unit.md`](unit.md).

Location: <!-- FILL -->
Naming: <!-- FILL -->
How to run just these: <!-- FILL -->

## What is real and what is faked

| Dependency | Real or faked | Why |
|---|---|---|
| The routing, serialization and middleware stack | Real | These tests exist to exercise it; bypassing it tests a handler that production never calls directly |
| Auth | Real mechanism, test credentials | Faking auth is how an endpoint ships unprotected while its tests are green |
| Persistence | <!-- FILL: real test database, or faked at the repository interface --> | <!-- FILL: whichever you choose, be consistent - mixing them makes failures ambiguous --> |
| Third-party services | Faked at your own client interface, or a recorded contract | Their availability is not your test's business. See [`mocking.md`](mocking.md) |

## What to assert

**On success:** the status code; the response body's shape, including field names exactly as
published; that no unexpected field leaked in (an extra field is a promise you did not mean to
make, and someone will start depending on it).

**On rejection:** the status code; the stable machine-readable error code; that the offending
field is identified; and that the message contains no internals - no stack traces, SQL, internal
hostnames, or raw upstream vendor errors.

**On auth:** unauthenticated is rejected; authenticated-but-unauthorized is rejected; and the
two are distinguishable only where that distinction is not itself a disclosure.

**On idempotency:** a replayed request with the same key returns the original result without
repeating the side effect, and a replay with the same key but different parameters is rejected
loudly. Skipping this test is how a timeout becomes a double charge.

**On pagination:** the first page, a middle page, the last page, and an empty result all behave;
the cursor or offset round-trips.

## What not to assert

- Exact human-readable message strings - they are meant to change. Assert the code.
- Field ordering in the payload.
- Response timing, unless you have an actual latency contract.
- Every business branch. Push those to unit tests, which run in milliseconds and pinpoint the
  failure far more precisely.

## The trap in this category

**Asserting only the status code.** `200 OK` with a payload missing half its fields passes a
status-only test, and the caller finds out in production. Assert the body's shape, always.

The mirror-image trap is asserting a full payload snapshot: it fails on every additive change,
so the team learns to regenerate snapshots without reading them, and the test stops being a
review of anything. Assert the fields that are part of the contract, deliberately.

## Compatibility regression

For each released version, keep a test that a caller written against the *original* contract
still succeeds. This is the only mechanism that reliably catches a field quietly renamed during
a refactor - see the checklist in
[`../../conventions-writing/patterns/api.md`](../../conventions-writing/patterns/api.md).

<!-- FILL: how you hold onto the old contract - recorded fixtures, a schema file per version,
     or a consumer-driven contract test. Name the mechanism and where the artifacts live. -->

## Example

```
test "creating an order twice with the same idempotency key charges once":
    given  a valid order payload and idempotency key K
    when   the request is sent twice
    then   both responses are 201 with the same order id
    and    exactly one charge exists
    and    a third request with key K and a different amount is rejected
           with code `idempotency.key_reuse`
```
