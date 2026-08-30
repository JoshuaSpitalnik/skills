# joshua-skills

A **template** skills repo. The structure is finished; the content is yours to fill in.

Four skills, each with its own reference tree, each triggering independently:

| Skill | Fires when | Answers |
|---|---|---|
| [`conventions-writing`](skills/conventions-writing/) | You are about to write or edit code | How do we name, shape, configure and log things here? |
| [`test-writing`](skills/test-writing/) | You are about to write or fix a test | What kind of test is this, and what does it assert? |
| [`systems`](skills/systems/) | A question spans more than one service | What owns this, what does it depend on, what breaks if it dies? |
| [`triage-structure`](skills/triage-structure/) | You got paged | This alert / symptom / error — where do I look first? |

Nothing in here describes a real system yet. Every ecosystem-specific fact is a
`<!-- FILL: ... -->` marker waiting for you.

---

## Install

```bash
# once, to register this repo as a marketplace
claude plugin marketplace add JoshuaSpitalnik/skills

# then
claude plugin install joshua-skills@joshuaspitalnik-skills
```

Working inside this repo, you can also point at the local checkout:

```bash
claude plugin marketplace add .
```

Skills are discovered by convention from `skills/`. Adding a fifth skill means adding a
directory with a `SKILL.md` — no manifest edit required.

---

## Layout

```
skills/
├── conventions-writing/
│   ├── SKILL.md                  router: task -> which pattern file
│   └── patterns/
│       ├── conventions.md        naming · structure · errors · logging
│       ├── anti-patterns.md      don't / why it bites / do instead
│       ├── api.md                request & response shape, versioning, errors
│       └── settings.md           config location, precedence, secrets
├── test-writing/
│   ├── SKILL.md                  router: what am I testing -> which pattern file
│   └── patterns/
│       ├── unit.md  api.md  repository.md  mocking.md
├── systems/
│   ├── SKILL.md                  router: which service doc to open
│   └── docs/
│       ├── internal-dependencies.md   the cross-service graph
│       └── <service>.md               one per service (copy _TEMPLATE.md)
└── triage-structure/
    ├── SKILL.md                  router: what do I have in hand?
    ├── guides/
    │   ├── quick.md              the first five minutes
    │   ├── field-reference.md    what each alert/incident field implies
    │   └── nested.md             cascading failures, upstream vs. downstream
    ├── indexes/
    │   ├── by-alert.md           alert name -> owning system
    │   ├── by-symptom.md         observed behavior -> ranked suspects
    │   └── by-error.md           error signature -> known cause
    ├── tools/
    │   ├── README.md             metrics / logs / traces / paging — which to ask
    │   ├── data-structure.md     the shape of what each returns
    │   └── scripts/CONTRACT.md   drop real query wrappers in here later
    └── evals/evals.json          skill-creator test scaffold
```

---

## How to fill it out

Three file conventions, used everywhere:

- **`_TEMPLATE.md`** — a blank skeleton. Copy it to add a new entry. Never fill it in place.
- **`_EXAMPLE.md`** — the same skeleton, filled in for a deliberately fictional subject
  (`orderflow`, a made-up service). It shows the intended density and tone. Delete it once your
  real entries make it redundant.
- **`<!-- FILL: ... -->`** — a marker in a real file saying what belongs there. Grep for it to
  see everything still outstanding:

  ```bash
  grep -rno "<!-- FILL" skills/ | wc -l     # how many are left
  grep -rl  "<!-- FILL" skills/            # which files are still stubs
  ```

  Two forms appear: `<!-- FILL: what goes here -->` in prose, and a bare `<!-- FILL -->`
  inside table cells where the column header already says what belongs there.

Leading underscores sort `_TEMPLATE.md` and `_EXAMPLE.md` above real content in every listing.

### Suggested order

1. **`systems/docs/`** first. Everything else links into it — triage indexes point at service
   docs, and a dependency graph is what makes cascade analysis possible.
2. **`triage-structure/indexes/`** next, starting with `by-alert.md`. Ten real alert rows are
   worth more than a perfect empty schema.
3. **`conventions-writing/patterns/`** as you notice yourself correcting the same thing twice.
4. **`test-writing/patterns/`** last — it is the least urgent and the easiest to derive from
   tests you already have.

### The rule that keeps this from rotting

**Facts live in exactly one file; everything else links.**

A service's owner, dependencies and failure modes live in `systems/docs/<service>.md` — and
nowhere else. A triage index row says *where to go*, never *what the fix is*. When you catch
yourself copying a fact into a second file, replace the copy with a link. Duplicated facts
diverge, and a triage index that lies is worse than one that is empty, because someone trusts
it at 3am.

---

## Testing a skill

Scaffolding for the official `skill-creator` eval loop lives at
`skills/triage-structure/evals/evals.json` (triage is the one skill here with objectively
checkable behavior: given an alert name, did it find the right system?).

The prompts are placeholders. Running the loop is only meaningful once the indexes hold real
rows — against an empty tree, with-skill and without-skill both produce nothing and the
benchmark is noise. Fill in a handful of alerts first, then ask Claude to run the evals.

---

## License

MIT. See [LICENSE](LICENSE).
