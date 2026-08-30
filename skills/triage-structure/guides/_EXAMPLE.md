# Handing over an open incident

> **Illustrative example.** Names, roles and timings here are fictional, written to show the
> intended shape and density of a triage guide. Nothing here describes a real process. Delete
> this file once your own guides make it redundant.

Handovers are where context dies. The person leaving holds a mental model built over two hours;
the person arriving gets a channel scrollback and a guess. Most of the second hour of a long
incident is the new responder re-deriving what the first one already ruled out.

## When to use this

At a shift boundary during an open incident, when escalating to a service owner, or when you
are too tired to be the one making decisions - which is itself a legitimate reason to hand over
and should not need justifying.

## When not to use this

For a resolved incident: that is a postmortem, and it has a different structure. For pulling in
an extra pair of hands while you stay in charge: that is not a handover, you stay the commander
and simply brief them on the one thing you want done.

---

## 1. Write the state before you talk

Speaking first and writing after means the written record ends up thinner than the conversation,
and the written record is what the third responder will read. Fill these five in the incident
channel:

- **Impact**: who cannot do what, since when, and whether the number is growing
- **Current theory**, with a confidence: "probably the 14:02 deploy, medium confidence"
- **Ruled out**, with how: the eliminations, and what evidence eliminated each
- **In flight**: what is currently running, who is running it, and when it should finish
- **Mitigations in place**: what has been changed, when, and whether it is safe to leave

## 2. Talk through it live

Five minutes, synchronous. The incoming responder asks questions; the outgoing one answers.
Written state cannot carry the hunches, and the hunches are often what shortens the next hour.

## 3. Transfer command explicitly

Say who has it now, in the channel, with a timestamp. An implicit handover produces two people
each assuming the other is deciding, which is worse than either of them deciding alone.

## 4. Stay reachable, briefly

Fifteen minutes for follow-up questions, then genuinely stop. An outgoing responder who lingers
undermines the incoming one's authority and delays their taking ownership.

---

## Common mistakes

**Handing over only the theory.** The theory is the least durable thing you have - it will
probably be wrong. The eliminations are what survive, and they are what the incoming responder
would otherwise spend an hour rediscovering.

**Handing over during an active mitigation.** Wait for it to land, or hand over the fact that it
is in flight with its expected completion time. A half-applied change with no owner is the
setup for the second incident.

**Not saying how confident you are.** "It's the deploy" gets acted on as fact; "probably the
deploy, but the timing is off by four minutes" gets verified. The second is more useful and
takes three more words.

## Links out

- If the incoming responder is starting cold: [`quick.md`](quick.md) from step 2
- If the theory involves an upstream service: [`nested.md`](nested.md)
- Escalation contacts per service: [`../../systems/docs/`](../../systems/docs/)
