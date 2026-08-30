# Settings and configuration

Configuration is the part of the system with no type checker, no test coverage by default, and
the shortest path to production. Most of these rules exist to make a misconfiguration fail
immediately and visibly rather than quietly at 2am under load.

## Where configuration lives

<!-- FILL: your sources, in order. A typical set:
     1. defaults in code        - safe for local development, never secret
     2. committed config files  - per-environment, non-secret
     3. environment variables   - deployment-specific overrides
     4. secret store            - credentials only, never in 1-3
     5. feature flags           - runtime-changeable behavior
-->

**Precedence, highest first:** <!-- FILL -->

Write the precedence down even if it seems obvious. Every config bug that takes more than an
hour is someone believing a different order than the code implements.

## Naming

<!-- FILL: your key naming rule, e.g. SCREAMING_SNAKE for env vars, prefixed with the service
     name, dot-separated in files: `ORDERFLOW__DB__POOL_SIZE`. Say how nesting maps between
     file keys and env vars, since that mapping is where typos hide. -->

- A key name says what it controls and its unit: `REQUEST_TIMEOUT_MS`, not `TIMEOUT`. A unitless
  duration is read as seconds by half your readers and milliseconds by the other half.
- Booleans are positive and unambiguous. `ENABLE_X=false` beats `DISABLE_X=false`, which nobody
  parses correctly on the first read.

## Validation

**Validate every setting at startup, not at first use.** A bad value discovered on the first
request is a bad value discovered in production, after the deploy was declared successful.

<!-- FILL: where startup validation happens and what it checks - presence, type, range, and
     mutual consistency between related keys. -->

- Fail to start on a missing required value. A service that boots in a broken state is worse
  than one that refuses to boot, because it takes traffic.
- Never fall back to a silent default for something security-relevant. A missing signing key
  should stop the process, not generate a random one.
- Log the resolved configuration at startup with secrets redacted. This single log line answers
  "what was it actually running with?" - which is the first question in a surprising number of
  incidents.

## Defaults

- Defaults are for development convenience and safe production behavior, in that order of
  frequency and the reverse order of importance.
- A default that is wrong in production is worse than no default, because nothing forces the
  conversation. If production must set it, make it required.
- Changing a default is a behavior change for every deployment that did not override it. Treat
  it like a breaking change: <!-- FILL: your process. -->

## Secrets

- Secrets never appear in source, committed config, log lines, error messages, or CI output.
- Secrets are referenced by name and resolved at runtime: <!-- FILL: from which store, and how
  a local developer gets a working value without holding a production one. -->
- **Rotation must be possible without a deploy,** or it will not happen. If rotating requires a
  release, it happens once and then never again.
- <!-- FILL: what to do when a secret leaks - who to tell, how to rotate, and where the runbook
  is. Write this before you need it. -->

## Feature flags

Flags are configuration that changes while the system is running, which makes them the fastest
way to change production behavior and the fastest way to break it.

<!-- FILL: your flag system, who can flip a flag, and whether a flip is audited. -->

- **Every flag needs an owner and a removal date at creation.** Flags with neither become
  permanent forks in the code, and the untested branch is the one that runs during your next
  incident.
- Default to the *current* behavior, so a flag system outage is a no-op rather than a
  simultaneous change everywhere.
- Do not branch on the environment name as a substitute for a flag - see
  [`anti-patterns.md`](anti-patterns.md). It makes staging stop being a test of production.

## Adding a new setting

- [ ] Name follows the rule above and states its unit
- [ ] Documented: what it controls, its type, its default, and the effect of changing it
- [ ] Validated at startup
- [ ] Safe default, or required with a startup failure
- [ ] Set in every environment before the code that reads it ships
- [ ] If secret, stored in the secret store and rotatable without a deploy
- [ ] If a flag, has an owner and a removal date
