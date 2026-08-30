# <!-- FILL: test category, e.g. "End-to-end tests" or "Property-based tests" -->

<!-- FILL: one or two sentences on what this category proves that the others cannot. If you
     cannot state that, the category probably should not exist - it will duplicate coverage
     and double the maintenance. -->

## Scope

Tests in this category cover: <!-- FILL -->
They deliberately do not cover: <!-- FILL: and say which category does instead -->

Location: <!-- FILL: path pattern -->
Naming: <!-- FILL: file and test-case naming -->
How to run just these: <!-- FILL: exact command -->

## What is real and what is faked

| Dependency | Real or faked | Why |
|---|---|---|
| <!-- FILL --> | <!-- FILL --> | <!-- FILL --> |

<!-- The "why" column is what keeps this honest. "Faked because it is slow" is a real reason;
     "faked because that is what we do" means nobody remembers the reason and the boundary
     will drift. -->

## Setup

<!-- FILL: how state is arranged, which helpers or factories exist, and how isolation between
     tests is guaranteed. Name the mechanism - "each test gets a fresh X" is only useful if the
     reader knows what enforces it. -->

## What to assert

<!-- FILL: the observable outcomes this category checks. -->

## What not to assert

<!-- FILL: the things that look testable here but belong elsewhere or nowhere. This section
     prevents more bad tests than the previous one. -->

## The trap in this category

<!-- FILL: the specific mistake people make here, and what it costs. Every test category has
     exactly one classic failure mode; naming it is worth more than the rest of the file. -->

## Example

<!-- FILL: one short, language-neutral sketch of a representative test. Structure and intent
     only - a reader should be able to translate it into the real framework without confusion. -->
