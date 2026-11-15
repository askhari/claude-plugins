# to-tasks — Brief (skill #5)

*Decided 2026-06-04 via `architect` (used meta, on the navigate system itself). Read
alongside `DESIGN.md`, `docs/to-prd-brief.md`, and `docs/architect-brief.md`.
Glossary terms live in DESIGN.md's Glossary section.*

## Problem

`to-prd` produces a durable, reviewable PRD (modules · user stories · testing decisions),
but an agent still can't *build* from it — a PRD is feature-shaped, not work-shaped. `to-tasks`
is the **layer slice**: it cuts ONE PRD into AFK/HITL **vertical-slice tracer-bullet** tasks
and writes them into the git-native `tasks/` queue the orchestrator drains. It is the second
half of the repeating **inner loop** (`to-prd → to-tasks`), and the skill that makes the
brief actually get built, one feature at a time.

**User:** the solo builder + their building agents (the orchestrator). **Slot:** pipeline
stage #5; follows `to-prd` (#4). Consumes `docs/prd/NNNN-slug.md` + CLAUDE.md/CONTEXT.md/ADRs
+ the codebase; writes `tasks/{open,claimed,done,needs-human}/NNNN-MM-slug.md`.

**Base concept:** vertical-slice tracer-bullet issues with AFK/HITL, retargeted from a GitHub
issue tracker to the **file task queue**, and made **idempotent-additive** (re-run = reconcile).

## Decided shape (the load-bearing calls)

### A. Operating shape — **fully autonomous; uncertainty → HITL tasks**

`to-tasks` is the most-unattended skill in the pipeline. It runs to completion with **no
blocking interview, ever** — a synchronous "quiz the user on granularity" seam is the
wrong tool for an AFK-first pipeline. Instead, every
channel for uncertainty is **externalized into the queue**, so the pipeline never pauses:

1. **Derive dependencies, don't ask them.** Read the PRD's `Modules` interface sketches: if
   module B's interface consumes module A's, B's slice is `blocked-by` A's. The dependency
   graph is *synthesized from the decomposition*, deterministically, with no human.
2. **Slice autonomously.** Tracer-bullet default — many thin vertical slices over few thick
   ones. `to-tasks` never stops to ask "is this granularity right?"
3. **Unresolvable doubt → a HITL *decision* task, not a blocking question.** When a slice
   needs a human call the PRD can't settle (ambiguous requirement, architectural fork, risky
   or irreversible step), `to-tasks` mints a **HITL task** and makes the dependent AFK slices
   `blocked-by` it. The orchestrator surfaces it async; the rest of the queue drains in
   parallel. This is the HITL-slice concept, made **async** instead of a synchronous
   quiz.

Ends with a **non-blocking summary** (slices created/changed/orphaned, the dependency graph,
any HITL decision tasks minted).

### B. Task template — **structured frontmatter + an issue-style body**

Store **only what's non-derivable and machine-read**; `id`, parent PRD, slice number, slug,
and status are all carried by the filename `NNNN-MM-slug.md` + its directory, so they are
**never** duplicated into frontmatter (the *derived parentage* ethos). PRD-membership
traceability **is** promoted to structured fields — it's genuinely useful for trace scripts
(`yq '.covers-stories'`, `grep '^modules:'`) and the reconcile keys off it — and it stays in
sync because reconcile regenerates it every run.

```
---
type: AFK | HITL              # label only — tells the orchestrator not to auto-dispatch HITL
covers-stories: [3, 4]        # → PRD user stories (traceability + reconcile key)
modules: [PaymentGateway]     # → PRD modules this slice touches (traceability + reconcile key)
blocked-by: [0007-01]         # task IDs; [] if none (uniform across AFK/HITL)
tests: true | false           # regenerated from PRD module `tested?`
scope: repo | portfolio       # HITL only; portfolio bubbles up
spec-hash: a1b2c3d4e5f6        # first 12 hex of sha256 over the slice's PRD projection
---
```

AFK body (issue-style, drops tracker mechanics):

```
PRD: docs/prd/0007-checkout.md      (convenience link; parent is authoritative via filename)

## What to build
End-to-end behavior of this vertical slice — cuts every layer (schema·API·UI·tests). No
file paths / code snippets (they go stale), EXCEPT a decision-encoding prototype snippet
(state machine / reducer / schema / type shape), trimmed to the decision-rich bits.

## Acceptance criteria
- [ ] ...
- [ ] tests (present iff `tests: true`)

## Covers
stories 3,4 · module: PaymentGateway      (human-readable echo of the frontmatter keys)
```

HITL **decision-task** body variant (describes a decision to make, not code):

```
## Decision needed
What a human must decide + why the PRD can't settle it.

## Context / options
The fork + tradeoffs to-tasks surfaced.

## Acceptance criteria
- [ ] Decision recorded (where)
- [ ] Dependent slices unblocked
```

`type`/`blocked-by` model (settled with `to-prd`'s reconcile and confirmed here): **blocking
is the universal mechanism, type is a label on top — not a directory distinction.** A blocker
blocks its dependents regardless of being AFK or HITL, so HITL tasks need no separate birth
lane. **All tasks are born in `open/`.** "Ready vs blocked" is **derived** (a task is ready
iff every `blocked-by` task is in `done/`), never a stored `blocked/` dir. Selection:
orchestrator drains `open/ ∧ type:AFK ∧ ready`; the human works `open/ ∧ type:HITL ∧ ready`
and watches `needs-human/`. `needs-human/` therefore means **escalation only** — not a birth
lane.

### C / D. Infra / cloud — **N/A**

A skill is a `SKILL.md`; nothing to host. Phases skipped (as with `to-prd`).

### E. Delivery & lifecycle — **emit into `tasks/`, always-on reconcile**

- **Numbering.** Slice number `MM` = next free integer scanning `tasks/*/NNNN-*` for the PRD,
  `max+1`, **never reused**. Parent `NNNN` comes from the PRD; slug from the slice intent.
- **Emission.** New tasks are written to **`open/`** only (both AFK and HITL). A plain file
  write — no tracker, no triage label.
- **Always-on reconcile.** Re-running `to-tasks NNNN` is *always* a **reconcile, not a
  regenerate** — there is no separate amend mode and `to-prd` sends no signal. It is the
  `tasks/` expression of *idempotent additive reconcile*.

## The reconcile (the contract, now OWNED here)

`to-prd` recorded this truth-table as a contract; `to-tasks` owns and implements it. On
`to-tasks NNNN` re-run:

| Existing task | Slice **unchanged** | Slice **changed** | Slice **gone** |
|---|---|---|---|
| *none (new)* | create `open/NNNN-MM` (MM = max+1) | — | — |
| `open/` | leave | **rewrite in place** (body + frontmatter + spec-hash) | surface orphan (report; no auto-delete) |
| `claimed/` *(in-flight)* | leave | **delta task** (don't mutate) | flag orphan → move to `needs-human/` |
| `done/` | leave | **new delta task** (don't re-open) | leave (history) |
| `needs-human/` | leave | update + re-flag | surface orphan |

**Invariants:** task ID is the **dedup key** (same slice → same `NNNN-MM`; new numbers
`max+1`, never reused) · **additive across the in-flight boundary** (`claimed`/`done` never
mutated — a change becomes a *delta task*; `infra-first / additive`) · **orphans surfaced,
not auto-deleted** · **no manifest** (tasks enumerated by glob, matched by coverage +
reasoning).

### Slice identity (the no-manifest match)

To apply the table, each regenerated slice must be matched to its existing `NNNN-MM` task
with no stored slice→task map. **Signature-first, fuzzy fallback:**

1. **Primary — coverage signature.** Match by `(covers-stories, modules)` read from
   frontmatter. Structural, glob + frontmatter scan, **no body reads**. A vertical slice's
   story-set usually discriminates it even when module-sets overlap.
2. **Fallback — fuzzy/reasoning, residuals only.** For tasks/slices left unmatched on both
   sides (coverage *drifted*: a rename, or a slice that now also covers story 5), a bounded
   pass — overlap score + slug + **one** body read — recognizes "same slice, shifted
   coverage" → **changed**, not orphan+new. Reasoning is the fallback, never the primary
   mechanism.

ID preserved on match; unmatched new slices get `max+1`; unmatched existing tasks are orphans.

### spec-hash (the cheap-reconcile lever)

Reconcile reads scale with the **change, not the feature**. Each task carries a `spec-hash`
in frontmatter; reconcile does a one-line `grep '^spec-hash:'`, recomputes, and compares —
**only a changed slice gets a body read/rewrite.**

**What is hashed (the crux): the PRD *source*, NOT the generated task prose.** `to-tasks`
synthesizes "What to build" with an LLM, so hashing that text would mark every slice changed
every run and defeat the lever. Instead the hash is over a **deterministic projection of the
PRD**, keyed by the slice's stored `covers-stories` + `modules`:

- **Projection = covered user-story lines + covered module blocks** (name · responsibility ·
  interface sketch · `tested?`), pulled **verbatim** from the current PRD, sorted (stories
  ascending, modules alpha) and whitespace-normalized.
- **Computed by a shell tool** — `… | shasum -a 256 | cut`, first 12 hex — **never** "eyeball
  a hash." Determinism requires a real hash function over exact text.
- **Per-slice, not whole-PRD** — a whole-PRD hash would flag all slices on any edit.

A changed story OR a changed module interface flips the hash of exactly the slices that cover
it; an unrelated PRD edit elsewhere does not. A one-module amend = ~one body read whether the
feature has 3 tasks or 30.

→ **Contract honored:** `to-tasks` MUST stamp every task with its slice's `spec-hash`.

### Delta-task identity (decided, not deferred)

When a *changed* slice's existing task is frozen (`claimed`/`done`), `to-tasks` mints a
**delta task** rather than mutate it:

- **Fresh `max+1` number**, born in `open/`, with the **current** coverage + spec-hash.
- **`amends: NNNN-MM`** frontmatter back-pointer to the frozen task it revises.
- **Supersession is DERIVED, not stored** (mirrors *derived parentage*). Before evaluating any
  frozen task as "changed," reconcile globs for `amends: <that-id>`; if a delta already points
  back at it, it's **already superseded → skip** (no second delta). This stops the
  infinite-delta chain **without writing to the frozen task** — honoring *never mutate frozen*.
  No `superseded-by` stamp on the original (that write would itself break the invariant).

## Roadmap (one-liners)

- **PRD-diff prune** — bound reconcile's blast radius with the amend's git diff before the
  hash scan (composes with the stamp; not required for the guarantee). *(Shared with `to-prd`.)*
- **Dependency-graph reconcile** — when slicing changes edges, re-point `blocked-by` across
  surviving tasks deterministically.
- **Delta-chain compaction** — fold a long `amends:` lineage into a readable history view.
- **Slug stability** — canonical slug derivation to make signature matches even cheaper.

## Out of scope

- Producing the PRD — that's `to-prd` (#4), upstream. `to-tasks` consumes it, never edits it.
- Deciding architecture/stack/modules — upstream (`architect`, `to-prd`).
- Writing code / claiming / merging — the **orchestrator** (the user's binary), downstream.
  `to-tasks` only *emits* tasks; it does not move them through `claimed`/`done`.
- A manifest of slices — explicitly rejected (glob + coverage signature + reasoning).

## Constraints

- **Fully autonomous** — no blocking interview; uncertainty becomes HITL tasks, dependencies
  are derived from module interfaces.
- **Idempotent additive reconcile, always-on** — never mutate `claimed`/`done`; emit delta
  tasks; surface (don't delete) orphans; no manifest.
- **Stamp every task with `spec-hash`** computed via a shell tool over the verbatim PRD
  projection (covered stories + module blocks), per-slice.
- **Derived everything that can drift** — readiness, parentage, and supersession are globbed,
  not stored.
- **Use the glossary vocabulary; respect ADRs** in the touched area.
- **Emit files into `tasks/`, not a tracker** — new tasks born in `open/`; `needs-human/` is
  escalation only.
