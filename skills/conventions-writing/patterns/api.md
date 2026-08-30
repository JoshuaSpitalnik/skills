# API conventions

The contract you publish is the hardest thing in the system to change, because you do not
control every caller and you often cannot see them. Everything here is about making the shape
predictable enough that a caller can guess correctly, and versioned enough that being wrong is
survivable.

Applies to: <!-- FILL: HTTP APIs, RPC, published events, and/or in-process public interfaces -
     say which, since the rules differ for each. -->

## Resources and operations

<!-- FILL: your resource naming and path shape, e.g. plural nouns, hyphenated,
     `/v1/orders/{order_id}/refunds`. State how nested a path may get before it should be a
     top-level resource - unbounded nesting is how paths become unusable. -->

| Operation | Shape | Notes |
|---|---|---|
| List | <!-- FILL --> | Always paginated - see below |
| Read one | <!-- FILL --> | |
| Create | <!-- FILL --> | Idempotency required - see below |
| Update | <!-- FILL: full replace, partial merge, or both --> | Say which, since a partial update with a `null` is ambiguous otherwise |
| Delete | <!-- FILL: hard or soft --> | |
| Action that is not CRUD | <!-- FILL: e.g. `POST /orders/{id}/cancel` --> | Prefer an explicit verb over a status field the caller must know to set |

## Payloads

- **Field naming** follows [`conventions.md` § Naming](conventions.md#naming) and does not
  change at the boundary. A field renamed on the way out means every conversation about it
  needs a translation step.
- **Never remove or repurpose a field** in a released version. Adding is safe; changing meaning
  is not, and is undetectable by the caller until it corrupts their data.
- **Absent, null, and empty are three different things.** Decide what each means and hold to it:
  <!-- FILL: e.g. absent = unchanged, null = explicitly cleared, [] = known-empty. -->
- **Enums are closed sets that will grow.** Document that callers must tolerate unknown values,
  or you cannot add one without a major version.

<!-- FILL: your envelope, if any - whether the resource sits at the top level or under a `data`
     key, and where metadata goes. Both work; mixing them does not. -->

## Pagination

<!-- FILL: cursor or offset, the parameter names, the default and maximum page size, and how
     the caller knows there is more. Prefer cursors for anything that changes while being
     paged - offset pagination silently skips and duplicates rows under concurrent writes. -->

## Idempotency

Any operation a client may retry needs a way to be safely repeated. Without it, a timeout the
client never saw becomes a double charge.

<!-- FILL: the header or field name, how long keys are retained, what happens on a replay with
     matching parameters vs. a replay with different parameters (the second is a client bug and
     should be rejected loudly, not silently accepted). -->

## Errors

The error envelope is part of the contract. Callers parse it, so it changes as carefully as
any success payload.

<!-- FILL: the exact shape, e.g.
     {
       "error": {
         "code": "order.already_captured",   // stable, machine-readable, never reworded
         "message": "...",                   // human-readable, safe to change
         "field": "amount",                  // present on validation failures
         "trace_id": "..."                   // so a support ticket maps to logs
       }
     }
-->

- **Codes are stable identifiers**, not prose. Once published, a code's meaning is frozen -
  callers branch on it.
- **Messages are for humans** and may change freely. Never make a caller parse one.
- **Include the `trace_id`** in every error response. It is the single thing that turns "the
  API returned an error yesterday" into a searchable incident.
- Status code mapping follows the error categories in
  [`conventions.md` § Errors](conventions.md#errors).
- **Never leak internals** - stack traces, SQL, internal hostnames, or upstream vendor errors
  passed through raw. They are reconnaissance for an attacker and noise for everyone else.

## Versioning and deprecation

<!-- FILL: where the version lives (path, header, media type), what counts as a breaking change
     in your ecosystem, and the deprecation process: how callers are notified, how long a
     version is supported, and how you find out who is still on it. That last one is the part
     usually missing - a deprecation you cannot measure is a deprecation you cannot finish. -->

## Compatibility checklist

Before publishing a change, confirm:

- [ ] No field removed, renamed, or given a new meaning
- [ ] No enum value removed; new values documented as expected
- [ ] No previously-optional request field made required
- [ ] No error code reworded or reassigned
- [ ] Defaults unchanged, or the change is deliberately breaking and versioned
- [ ] <!-- FILL: your own additions -->
