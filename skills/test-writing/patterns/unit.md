# Unit tests

The fast, numerous layer. A unit test proves one piece of logic behaves correctly across its
interesting inputs, and it does so quickly enough that you run the suite without thinking about
it. Speed is a feature here: a unit suite that takes five minutes stops being run before a
commit, and a suite that is not run before a commit is a suite that finds bugs late.

## Scope

Tests in this category cover: pure logic - calculations, validation, state transitions,
parsing, branching rules.
They deliberately do not cover: anything crossing a process boundary. Database access belongs
in [`repository.md`](repository.md); wire contracts belong in [`api.md`](api.md).

Location: <!-- FILL -->
Naming: <!-- FILL -->
How to run just these: <!-- FILL -->

If a unit test needs a database, a network, a filesystem, or a real clock, either the test is
in the wrong category or the unit has a dependency it should be receiving rather than creating.

## What is real and what is faked

| Dependency | Real or faked | Why |
|---|---|---|
| The unit under test | Real | Obviously |
| Pure collaborators (value objects, other pure functions) | Real | Faking them tests the fake and hides real integration mistakes between your own pure code |
| I/O collaborators (stores, clients, queues) | Faked at an interface you own | Keeps the test fast and deterministic. See [`mocking.md`](mocking.md) |
| Clock, randomness, unique IDs | Injected | Otherwise the interesting cases are untestable and the boring ones are flaky |

<!-- FILL: your convention for injecting the clock and randomness. Whatever it is, having one
     is what makes time-dependent logic testable at all. -->

## What to assert

The returned value or resulting state, for each meaningful class of input:

- The ordinary case, so the test documents the intended behavior.
- Each **boundary**: empty, one, many; zero, negative, maximum; just inside and just outside
  every threshold. Off-by-one errors live exactly here, and nowhere else.
- Each **branch** the logic can take, including the ones that only fire on bad input.
- Each **error condition**: assert the specific error and its message, not merely that
  something was raised. "It threw" passes even when it threw for the wrong reason - typically a
  typo in the test's own setup.

## What not to assert

- Which internal helpers were called, or in what order. That is the implementation, and it
  should be free to change. See [`mocking.md`](mocking.md).
- Log output or metric emission, unless the log line *is* the deliverable.
- Formatting of a message that has no contract - it will change, the test will fail, and
  nothing will have been protected.

## The trap in this category

**Mirroring the implementation in the assertion.** When a test computes its expected value with
the same expression the code uses, it passes for any bug that lives in that expression - the
test and the code are wrong together. Write expected values as literals, worked out by hand or
taken from a known-good source. A hand-computed constant is the only assertion that can
disagree with the code.

The related trap is testing the language rather than your logic: a test that a getter returns
what a setter set proves the runtime works.

## Table-driven cases

Where the same behavior holds across many inputs, a table keeps the boundaries visible and the
count of cases honest:

```
cases:
  | input        | expected | note                       |
  | 0            | 0        | boundary: nothing to do    |
  | 1            | 1        | boundary: single           |
  | 99           | 99       | just inside the limit      |
  | 100          | error    | at the limit               |
  | -1           | error    | boundary: negative         |
for each case: assert transform(input) == expected
```

Each row must carry its own identifying label in the failure output. A table test that reports
"case 4 failed" makes you count rows.

## Example

```
test "a refund larger than the original charge is rejected":
    given  a charge of 100.00 EUR
    when   a refund of 100.01 EUR is requested
    then   the result is an error with code `refund.exceeds_charge`
    and    the error names the maximum refundable amount
```
