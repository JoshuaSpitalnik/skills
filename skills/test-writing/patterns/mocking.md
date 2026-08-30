# Mocking and fixtures

Every fake is a claim that something behaves a certain way. When the claim is wrong, the test
passes and production does not - and that failure mode is silent, so the suite gets *more*
trusted as it drifts further from reality. This file is about keeping the claims few, honest,
and located where you can notice them going stale.

## The rule

**Fake at boundaries you own; use the real thing everywhere else.**

An interface you defined is one you control: if it changes, you change the fake in the same
commit, and the compiler or the test suite tells you when you forget. A third-party library's
internals, your own pure functions, or the database are not boundaries you own - faking them
means maintaining a model of something that changes without telling you.

| Dependency | Default | Why |
|---|---|---|
| Your own pure logic | Real | It is fast and deterministic already; a fake tests the fake and hides real mistakes between your own modules |
| Your own I/O interface (store, client, publisher) | Fake | Fast, deterministic, and you own the shape, so drift is visible |
| The database, in data-access tests | Real | Constraints and coercion do not exist in a fake. See [`repository.md`](repository.md) |
| Third-party HTTP services | Fake, at *your* client interface | Their availability and rate limits are not your test's business |
| Third-party libraries' internals | Real | Faking these couples your tests to another team's refactors |
| Clock, randomness, ID generation | Inject | Not really mocking - it is making non-determinism a parameter |

<!-- FILL: your fake style - hand-written stubs, generated doubles, an in-memory implementation
     of each interface - and where they live. In-memory implementations age better than
     per-test stubs: there is one of each, so when reality moves you fix one file. -->

## Verify outcomes, not interactions

The default assertion is on **state and returned values**. Asserting that a collaborator's
method was called with particular arguments couples the test to the current implementation:
rename the method, split it in two, or reorder two calls, and the test fails while the behavior
is unchanged. A suite full of interaction assertions makes refactoring expensive, which is the
opposite of what tests are for.

The exception is when **the call itself is the behavior**. Sending the email *is* the outcome of
"notify the customer"; publishing the event *is* the outcome of "order confirmed". There is no
state to inspect, so assert the interaction - and assert it once, at the level that owns the
behavior.

**Never assert the mock's own configuration.** A test that stubs a client to return `X` and then
asserts the client returns `X` has tested the mocking library.

## Keep fakes honest

A fake diverges from reality quietly. Three things keep the gap small:

- **One fake per interface, shared** - not a fresh stub per test. When reality changes there is
  one place to fix, and one place to look when something is suspicious.
- **Make fakes enforce the real contract.** If the real client rejects an empty ID, the fake
  should too. A permissive fake lets your code develop a dependency on behavior the real thing
  does not offer.
- **Cover the failure modes, not just success.** Timeouts, rate limits, malformed responses,
  partial results, slow responses. These are what the real dependency does on its worst day,
  and your handling of them is untested until the fake can produce them.

<!-- FILL: for each third-party you depend on, where the recorded or contract-tested version of
     its real responses lives, and how often it is refreshed. A fake nobody has compared against
     reality in a year is a guess. -->

## Fixtures

Fixture data is test input, and test input should be legible from the test.

- **Prefer factories with explicit overrides** over shared static fixtures. `an_order(status:
  refunded)` states what matters and hides what does not. A shared `orders.json` requires the
  reader to go find out which row they got and hope nobody edits it.
- **Every value that the assertion depends on is set in the test**, not inherited from a
  default. When a reader cannot tell why a test expects `42`, the test has stopped documenting
  anything.
- **Randomize the values that should not matter,** and fix the ones that should. Random
  irrelevant data catches accidental dependencies on a particular string; random *relevant*
  data produces flakes.

## The trap in this category

**Mocking the thing you are trying to test.** It happens gradually: a unit needs three
collaborators, each is faked, and eventually the test asserts that a sequence of fakes were
called in order. It passes forever and catches nothing, because none of the real code runs.

If a test needs more than a couple of fakes, that is the design talking, not the test. The
usual fix is not a better mocking setup - it is a smaller unit with fewer dependencies, which
[`unit.md`](unit.md) will then cover directly.

## Example

```
test "a failed notification does not roll back the confirmed order":
    given  a notification client faked to raise a timeout
    when   an order is confirmed
    then   the order is persisted as confirmed        <- state: the real outcome
    and    a retry was enqueued                        <- state, not "retry() was called"
    and    one notification attempt was made           <- interaction: sending IS the behavior
```
