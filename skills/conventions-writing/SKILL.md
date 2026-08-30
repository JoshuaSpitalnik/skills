---
name: conventions-writing
description: House coding conventions for this ecosystem - naming, module structure, API shape, settings and config handling, error handling, logging, and the anti-patterns to avoid. Use this skill whenever writing, editing, reviewing, refactoring, or generating code in this ecosystem, before choosing a name, deciding where a new file goes, designing an endpoint or function signature, adding a config key, or writing a log line - even when the user never mentions conventions, style, or standards.
---

# Writing code in this ecosystem

Conventions exist so a reader can predict where things are and how they behave without
reading everything. That is the whole payoff: a name that follows the rule tells you what a
thing is; a log line that follows the rule can be found at 3am by someone who has never seen
the code.

Consult this before you write, not after. Renaming and relocating a file after review is
cheap in effort and expensive in attention.

## Which file to read

Read the one that covers your decision. They are separate files so that a naming question
does not drag API and testing rules into context.

| You are about to... | Read |
|---|---|
| Name a file, type, function, variable, constant, or branch | [`patterns/conventions.md`](patterns/conventions.md) § Naming |
| Decide where a new file or module goes, or add an import across a layer | [`patterns/conventions.md`](patterns/conventions.md) § Structure |
| Raise, return, wrap, or swallow an error | [`patterns/conventions.md`](patterns/conventions.md) § Errors |
| Add or change a log line | [`patterns/conventions.md`](patterns/conventions.md) § Logging |
| Design or change an endpoint, payload, or public interface | [`patterns/api.md`](patterns/api.md) |
| Add a config key, env var, feature flag, or secret | [`patterns/settings.md`](patterns/settings.md) |
| Review code, or reach for something that feels clever | [`patterns/anti-patterns.md`](patterns/anti-patterns.md) |

Adding a whole new category of convention? Copy [`patterns/_TEMPLATE.md`](patterns/_TEMPLATE.md).
[`patterns/_EXAMPLE.md`](patterns/_EXAMPLE.md) shows a filled-in one.

## How to apply a rule

**Cite the rule you followed** when a choice is non-obvious - in the PR description, or a short
comment where the code would otherwise look arbitrary. A reviewer who can see which rule you
applied can disagree with the rule instead of with you, and rules are easier to change than
habits.

**Match the surrounding code when it conflicts with a rule.** A file written before a
convention existed is internally consistent, and half-migrating it is worse than either state.
Note the mismatch, follow the local pattern, and migrate the file separately if it matters.

**When two rules collide,** prefer in this order: correctness, then the reader's ability to
find things (naming and structure), then brevity. If a collision recurs, that is a signal the
rules need editing - say so rather than picking silently each time.

**When no rule covers your case,** pick the option most consistent with the nearest analogous
code, and add the rule to the relevant pattern file in the same change. A convention repo that
only grows when someone remembers to update it never grows.

## What this skill does not decide

Test structure lives in the `test-writing` skill. Which service owns a behavior lives in
`systems`. If you find yourself writing either of those here, link instead.

<!-- FILL: any ecosystem-wide rule that is too short to deserve its own pattern file -
     e.g. "all timestamps are UTC ISO-8601", "no ambient global state", "public functions
     documented, private ones only where non-obvious". Keep this list under ten lines; when
     it grows past that, promote entries into patterns/conventions.md. -->
