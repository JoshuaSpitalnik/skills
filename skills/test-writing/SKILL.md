---
name: test-writing
description: How to write tests in this ecosystem - unit tests, API and contract tests, repository and data-access tests, and the mocking and fixture policy. Use this skill whenever adding, changing, reviewing, debugging, or fixing tests, whenever a code change needs coverage, whenever a test is flaky or slow, and before deciding where a test goes, what it is named, how it is arranged, or what should be mocked - even when the user only says "add a test" or "write tests for this".
---

# Writing tests in this ecosystem

A test earns its place by failing for exactly one reason and saying what that reason is. Tests
that fail for many reasons get muted; tests that fail without explaining themselves get
deleted by whoever is unlucky enough to be on the failing build.

Decide what you are testing before you decide how, because the answer determines which rules
apply.

## Which file to read

| What you are testing | Read |
|---|---|
| A function, rule, or calculation in isolation | [`patterns/unit.md`](patterns/unit.md) |
| An endpoint, handler, or published contract | [`patterns/api.md`](patterns/api.md) |
| Queries, persistence, migrations, transactions | [`patterns/repository.md`](patterns/repository.md) |
| Anything where you are deciding what to fake | [`patterns/mocking.md`](patterns/mocking.md) |

Adding a new category of test? Copy [`patterns/_TEMPLATE.md`](patterns/_TEMPLATE.md);
[`patterns/_EXAMPLE.md`](patterns/_EXAMPLE.md) shows one filled in.

Nearly every test also touches mocking. Read that file alongside whichever other one applies -
the most common review comment on a new test is that it mocks the wrong thing.

## Rules that apply to every test

**Name the test after the behavior and its condition,** not after the function. A reader
scanning a failure summary should learn what broke without opening the file:
`rejects_a_refund_larger_than_the_original_charge` tells you something;
`test_refund_2` does not.

<!-- FILL: your naming convention and where test files live - beside the source, or in a mirror
     tree. Give one concrete path example; it settles more arguments than a paragraph. -->

**One behavior per test.** Several assertions are fine when they describe one outcome. Several
*scenarios* in one test means the first failure hides the rest, and you fix them one build at
a time.

**Arrange, act, assert - and keep the arrange short.** A test whose setup is thirty lines is
telling you the unit under test has too many dependencies. That is a design signal worth
hearing rather than working around with a bigger fixture.

**Assert on the observable outcome, not on how it was reached.** A test that asserts which
internal methods were called fails on every refactor while catching no bugs. It converts your
test suite from a safety net into a tax on change. See
[`patterns/mocking.md`](patterns/mocking.md).

**Make the failure message do the work.** When an assertion fails, the output should name the
expected and actual values in domain terms. If it does not, the next person spends their time
re-deriving what the test meant.

**No test depends on another test, on ordering, on wall-clock time, or on the network.** Each
of these produces failures that appear unrelated to the change that triggered them, and
unrelated failures are how a suite loses its credibility.

**Fix flakes, do not sleep on them.** A `sleep` turns a race into a slower, rarer race that
now also costs build minutes. Wait on the condition, or fix the race. A test muted "for now"
is a test deleted with extra steps.

<!-- FILL: your test runner, how to run one test vs. the whole suite, and any coverage
     expectation. Say what the number is for - a coverage target that nobody can explain gets
     satisfied by tests that assert nothing. -->

## What this skill does not decide

Naming of production code, module layout and logging live in `conventions-writing`. Which
service owns a behavior lives in `systems`.
