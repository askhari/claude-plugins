# to-prd — Brief (skill #4)

*Decided 2026-06-03 via `architect` (used meta, on the navigate system itself). Read
alongside `DESIGN.md`, `docs/architect-brief.md`, and `docs/scaffold-skills-brief.md`.
Glossary terms live in DESIGN.md's Glossary section.*

## Problem

The brief (CLAUDE.md + CONTEXT.md) decides the whole project — what/who/why, architecture,
stack. But an agent can't build from it directly: it stops at architecture altitude,
naming roadmap items but not the **modules, user stories, and testing decisions** one
feature needs. `to-prd` is the **feature zoom** — it takes ONE roadmap item to a durable,
reviewable PRD that `to-tasks` can slice. It completes the `what → architecture → tech →
feature` fractal.

**User:** the solo builder + their building agents. **Slot:** pipeline stage #4; the first
half of the repeating **inner loop** (`to-prd → to-tasks`) that realizes the brief one
feature at a time. Consumes CLAUDE.md/CONTEXT.md/ADRs + the roadmap item + the codebase;
writes `docs/prd/NNNN-slug.md` (the **feature ledger**).

## Decided shape (the load-bearing calls)

### A. Operating shape — **thin hybrid (synthesize-first + one module seam)**

The upstream brief is already rich, so a full relentless interview would re-ask what's
settled; but the **deep-module decomposition** is the one thing the brief does *not*
contain (it's finer than architecture) and the least-recoverable call in a PRD. So:

1. Read brief + glossary + ADRs + the roadmap item; explore the codebase.
2. **Synthesize** the draft PRD (problem, solution, user stories, out-of-scope) silently —
   the brief grounds it.
3. **Interview seam** — present the deep-module decomposition: *do these modules match
   expectations? which modules get tests?* Iterate at this one seam until approved.
4. Write `docs/prd/NNNN-slug.md`.

- *Not* a pure synthesizer — modules are too load-bearing to guess.
- *Not* a full feature-zoom interview (`navigate-brainstorm`/`architect` machinery) — that
  re-asks what the rich brief already decided.

The seam is exactly one interactive checkpoint, kept; everything
else is synthesized.

### B. PRD template — **`Modules` promoted to first-class**

Module decomposition is now the interview seam AND `to-tasks`' slice source, so it gets its
own top-level section instead of being buried in a bullet list. The base's "publish to
issue tracker + `ready-for-agent` label" is dropped — navigate emits a file (phase E).

```
## Problem
## Solution
## User Stories       (numbered, extensive)
## Modules            ← first-class: name · responsibility · interface sketch · tested? + why
## Other Decisions    (schema / API / arch notes that aren't module-shaped)
## Testing Decisions  (what makes a good test ; which modules ; prior art)
## Out of Scope
## Further Notes
```

Deep module = lots of functionality behind a simple, testable interface that rarely
changes. Use the project's glossary vocabulary; respect ADRs in the touched area.

### C / D. Infra / cloud — **N/A**

A skill is a `SKILL.md`; nothing to host. Phases skipped.

### E. Delivery & lifecycle — **file-ledger emission, amend-in-place**

- **Numbering:** `NNNN` = next free zero-padded 4-digit integer scanning `docs/prd/` —
  same convention as `docs/adr/0001-` and task IDs (`0007`). Slug from the roadmap item.
- **Unit:** one PRD per roadmap item (that *is* the feature zoom).
- **Emission:** a plain file write + a handoff line to `to-tasks`. No tracker, no triage
  label (those were the base's tracker mechanics).
- **Amend-in-place:** re-invoking `to-prd` on an existing feature edits the **same**
  `NNNN-slug.md`; **git is the history**. One stable number per feature forever — which
  `derived parentage` depends on (`tasks/*/0007-*` must keep resolving to PRD `0007`).
  The module seam re-runs so decomposition stays current. No provenance stamp on the PRD
  itself (it's wholly `to-prd`-owned and reviewed by diff — unlike `.claude/`).
  - *Rejected — append-only/supersede (revision → new `0012`):* orphans every existing
    `0007-*` task from the live PRD and forces a re-pointing pass. *Rejected — create-only:*
    no tool-supported path to re-slice an evolving feature.

## Cross-skill contract — the `to-tasks` reconcile (recorded here, owned by `to-tasks`)

Amend-in-place is only safe because **`to-tasks`' re-run is a reconcile, not a regenerate**
— the `tasks/` expression of `idempotent additive reconcile` (the principle DESIGN.md
already coined for `scaffold-skills`' `.claude/`). On `to-tasks 0007` re-run:

| Existing task | Slice unchanged | Slice **changed** | Slice **gone** |
|---|---|---|---|
| *none (new)* | create `open/0007-NN` (NN = max+1) | — | — |
| `open/` | leave | rewrite body in place | surface orphan |
| `claimed/` *(in-flight)* | leave | **delta follow-up task** (don't mutate) | flag orphan → `needs-human` |
| `done/` | leave | **new delta task** (don't re-open) | leave (history), note superseded |
| `needs-human/` | leave | update + re-flag | surface orphan |

Invariants: **task ID is the dedup key** (same slice → same `0007-NN`; new numbers `max+1`,
never reused) · **additive across the in-flight boundary** (`claimed`/`done` never mutated
— a change becomes a *delta task*; `infra-first / additive`) · **orphans surfaced, not
auto-deleted** · **no manifest** (tasks enumerated by glob, matched by reasoning on content).
`to-tasks` is *always* idempotent-additive — there's no separate amend mode and `to-prd`
sends no signal.

**Cheap reconcile (the cost lever).** Reads scale with the *change*, not the feature:

1. **Glob for ID-set cases** — "new" and "orphan" detection need only `ls tasks/*/0007-*`
   (filenames carry ID + slug; directory carries status). **Zero body reads.**
2. **Only `open/` is rewritten** — `claimed`/`done` are never mutated, so their bodies are
   never opened to edit.
3. **`spec-hash` stamp** — each task carries a frontmatter hash of the slice spec it came
   from (the `tasks/` instance of the **provenance stamp**). Reconcile does a one-line
   header scan (`grep '^spec-hash:'`) to tell changed from unchanged; **only changed slices
   get a body read.** A one-module amend = ~one body read whether the feature has 3 tasks
   or 30.

→ **Contract imposed on `to-tasks`:** it MUST stamp each task with its slice's `spec-hash`.
The full truth-table + cost model belong to the eventual `to-tasks` brief; recorded here
because amend-in-place depends on them.

## Roadmap (one-liners)

- **PRD-diff prune** — bound reconcile's blast radius using the amend's git diff before the
  hash scan (composes with the stamp; not required for the guarantee).
- **Delta-task identity** — exact convention for a `claimed`/`done` follow-up task (number +
  back-pointer to the slice it amends).
- **Seam depth** — when (if ever) the synthesize step itself wants a light confirm beyond
  the module seam (e.g. ambiguous user stories).
- **Glossary-drift check** — verify PRD vocabulary against CONTEXT.md at write time.

## Out of scope

- Slicing the PRD into tasks — that's `to-tasks` (#5), downstream. `to-prd` produces the
  durable parent; it does NOT move through task dirs.
- The full reconcile truth-table / `spec-hash` format — `to-tasks` owns it (contract noted
  above).
- Deciding architecture/stack — that's `architect` (#2), upstream.
- A whole-project PRD — `to-prd` is per-feature by design (a whole-project PRD goes stale at
  feature #2).

## Constraints

- **Synthesize-first, interview only at the module seam** — do not re-ask what the brief
  settled; do not guess the module decomposition.
- **Durable & reviewable** — the PRD is a stable artifact under git; **amend-in-place**,
  never renumber a living feature (derived parentage depends on the stable number).
- **Use the glossary vocabulary; respect ADRs** in the touched area.
- **Emit a file, not a tracker issue** — `docs/prd/NNNN-slug.md` + a handoff line to
  `to-tasks`.
- **Stable-number contract** — because `to-tasks` reconciles by `0007-*`, `to-prd` must keep
  the number across amends.
