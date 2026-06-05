---
name: to-tasks
description: Layer slice — cut ONE PRD into AFK/HITL vertical-slice tracer-bullet tasks and write them into the git-native tasks/ queue the orchestrator drains. Fully autonomous (no blocking interview — uncertainty becomes HITL tasks); re-runs are an idempotent additive reconcile against the amended PRD. Fifth skill in the navigate pipeline; follows `to-prd`, hands off to the orchestrator. Use to slice a feature into tasks, or on "to-tasks", "slice this PRD", "tasks for <PRD>".
---

<what-to-do>
Cut ONE PRD into **vertical-slice tracer-bullet** tasks and emit them into `tasks/`. You are
the **layer slice** — finer than the PRD (feature-shaped), the work an agent actually builds.
Run **fully autonomously: no blocking interview, ever.** The pipeline is AFK-first, so every
channel for uncertainty is externalized into the queue, never raised as a synchronous prompt:

- **Derive dependencies** from the PRD's module interfaces (don't ask them).
- **Slice autonomously** — tracer-bullet default, many thin slices over few thick ones.
- **Unresolvable doubt → a HITL *decision* task** that blocks its dependent AFK slices — not a
  blocking question. The orchestrator surfaces it async; the rest of the queue drains.

Re-running on the same PRD is **always a reconcile, not a regenerate** (see `<reconcile>`).
Stay at TASK altitude: vertical slices through every layer. Do NOT edit the PRD (upstream),
do NOT write code or claim/merge (the orchestrator, downstream). Use the project's glossary
vocabulary; respect ADRs in the touched area. Coin/record any new term in CONTEXT.md.
</what-to-do>

<inputs>
Read before slicing anything:

- **The PRD** — `docs/prd/NNNN-slug.md`, passed (or named) as the argument. The ONE feature
  you slice. Its `Modules` (decomposition + interface sketches) and `User Stories` are the
  slice source; `tested?` flags drive test slices.
- **CLAUDE.md / CONTEXT.md** — brief + glossary; use the vocabulary, honor the architecture.
- **`docs/adr/`** — respect existing decisions in the area you're touching.
- **The codebase** — explore the current state of the modules you'll slice through.
- **`tasks/*/NNNN-*`** — the feature's existing tasks (if any). Their presence means this is a
  **reconcile** (see `<reconcile>`), and they give you the next free `MM` (`max+1`).

If the PRD argument is ambiguous or absent, ask which PRD to slice — that one question only.
Everything else is autonomous.
</inputs>

<process>
1. **Synthesize the slice set silently.** From the PRD + codebase, draft **vertical
   tracer-bullet** slices — each a thin but COMPLETE path through every layer (schema · API ·
   UI · tests), demoable on its own. Prefer many thin slices. For each slice fix its
   **coverage** (`covers-stories` + `modules`) and whether it carries tests (from the covered
   modules' `tested?`).

2. **Derive the dependency graph.** From the `Modules` interface sketches: if a slice's module
   consumes another module's interface, it is `blocked-by` that module's slice. Express edges
   as `blocked-by: [task-ids]`. Keep it a DAG.

3. **Mint HITL decision tasks for unresolvable doubts.** Where a slice needs a human call the
   PRD can't settle (ambiguous requirement, architectural fork, risky/irreversible step),
   create a `type: HITL` task describing the decision, and make the dependent AFK slices
   `blocked-by` it. Never block the run to ask — externalize it.

4. **Reconcile against existing tasks** (always; see `<reconcile>`). Match each slice to its
   existing `NNNN-MM` task, apply the truth-table, stamp `spec-hash`.

5. **Write** the resulting tasks into `tasks/open/NNNN-MM-slug.md` (new + rewritten), emit
   delta tasks, and flag orphans per the table. Then **report** a non-blocking summary and
   hand off to the orchestrator.
</process>

<task-template>
Store ONLY non-derivable, machine-read fields in frontmatter — `id`/parent/slice/slug come
from the filename `NNNN-MM-slug.md`, status comes from the directory; never duplicate them.

```
---
type: AFK | HITL              # label only — orchestrator does not auto-dispatch HITL
covers-stories: [3, 4]        # → PRD user stories (traceability + reconcile key)
modules: [PaymentGateway]     # → PRD modules this slice touches (traceability + reconcile key)
blocked-by: [0007-01]         # task IDs; [] if none (uniform across AFK/HITL)
tests: true | false           # from the covered modules' `tested?`
scope: repo | portfolio       # HITL only; portfolio bubbles up
spec-hash: a1b2c3d4e5f6        # first 12 hex of sha256 over the slice's PRD projection
---
PRD: docs/prd/0007-checkout.md      (convenience link; parent is authoritative via filename)

## What to build
End-to-end behavior of this vertical slice — cuts every layer. NO file paths or code snippets
(they go stale), EXCEPT a decision-encoding prototype snippet (state machine / reducer /
schema / type shape), trimmed to the decision-rich bits and noted as prototype-derived.

## Acceptance criteria
- [ ] ...
- [ ] tests pass (present iff `tests: true`)

## Covers
stories 3,4 · module: PaymentGateway      (human echo of the frontmatter keys)
```

HITL **decision-task** body variant (a decision to make, not code to build):

```
## Decision needed
What a human must decide + why the PRD can't settle it.

## Context / options
The fork + tradeoffs you surfaced.

## Acceptance criteria
- [ ] Decision recorded (where)
- [ ] Dependent slices unblocked
```

**Type / blocking model.** Blocking is the universal mechanism; `type` is a label on top, not
a directory. All tasks — AFK and HITL — are **born in `open/`**. "Ready vs blocked" is
**derived** (ready iff every `blocked-by` task is in `done/`), never a stored dir. `needs-human/`
is **escalation only**, not a birth lane.
</task-template>

<spec-hash>
Stamp every task with `spec-hash` — the lever that makes reconcile reads scale with the
**change, not the feature**. Hash the **PRD source, NOT the generated task prose** (prose
varies run-to-run and would mark everything changed). Procedure, per slice:

1. From the slice's `covers-stories` + `modules`, pull the **verbatim** PRD fragments: each
   covered user-story line, and each covered module block (name · responsibility · interface
   sketch · `tested?`).
2. Canonicalize: stories sorted ascending, modules sorted alpha, whitespace-normalized,
   joined with a stable delimiter.
3. **Compute with a shell tool** — pipe the canonical projection to `shasum -a 256` and take
   the first 12 hex chars. Never hand-compute a hash.

This is **per-slice** (a whole-PRD hash would flag every slice on any edit). A changed story
or module interface flips exactly the slices that cover it; an unrelated edit elsewhere does
not.
</spec-hash>

<reconcile>
Re-running `to-tasks NNNN` is **always a reconcile, not a regenerate** — there is no amend
mode and `to-prd` sends no signal. It is the `tasks/` expression of *idempotent additive
reconcile*.

**Match each slice to its existing task — signature-first, fuzzy fallback (no manifest):**
1. **Primary — coverage signature.** Match regenerated slice ↔ existing task by
   `(covers-stories, modules)` from frontmatter. Glob `tasks/*/NNNN-*` + scan frontmatter;
   **no body reads**.
2. **Fallback — fuzzy/reasoning, residuals only.** For tasks/slices unmatched on both sides
   (coverage drifted — rename, or now also covers another story), do a bounded pass (overlap
   score + slug + ONE body read) to recognize "same slice, shifted coverage" → **changed**,
   not orphan+new.

**Then `grep '^spec-hash:'`, recompute (see `<spec-hash>`), compare — only a CHANGED slice
gets a body read/rewrite.** Apply the truth-table:

| Existing task | Slice **unchanged** | Slice **changed** | Slice **gone** |
|---|---|---|---|
| *none (new)* | create `open/NNNN-MM` (MM = max+1) | — | — |
| `open/` | leave | **rewrite in place** (body + frontmatter + `spec-hash`) | surface orphan (report; no delete) |
| `claimed/` *(in-flight)* | leave | **delta task** (don't mutate) | flag orphan → move to `needs-human/` |
| `done/` | leave | **new delta task** (don't re-open) | leave (history) |
| `needs-human/` | leave | update + re-flag | surface orphan |

**Invariants:** task ID = **dedup key** (same slice → same `NNNN-MM`; new numbers `max+1`,
never reused) · `claimed`/`done` **never mutated** (additive across the in-flight boundary) ·
orphans **surfaced, not deleted** · **no manifest**.

**Delta tasks** (for a changed slice whose task is `claimed`/`done`): a fresh `max+1` task in
`open/`, with the **current** coverage + `spec-hash` and an **`amends: NNNN-MM`** back-pointer.
**Supersession is derived, not stored:** before evaluating any frozen task as "changed", glob
for `amends: <that-id>`; if a delta already points back at it, it's **already superseded →
skip** (no second delta, no write to the frozen task).
</reconcile>

<outputs>
- Writes tasks into **`tasks/open/NNNN-MM-slug.md`** (new + rewritten + delta), born in `open/`.
- On reconcile: moves `claimed/` orphans → `needs-human/`; updates `needs-human/` changed
  tasks in place; leaves `claimed`/`done` otherwise untouched.
- Records any new term in **CONTEXT.md**.
- Reports a tight, **non-blocking** summary: slices created / changed / orphaned, the
  dependency graph, and any HITL decision tasks minted (with what they block).
- **No ADR** — task slicing is downstream of architecture; ADRs belong to `architect`.

Hand off to the **orchestrator** (the user's binary) — it drains `open/ ∧ type:AFK ∧ ready`;
the human works `open/ ∧ type:HITL ∧ ready` and `needs-human/`.
</outputs>

<supporting-info>
## Altitude
Task slicing — vertical tracer-bullet slices through every layer. NOT product scope
(`navigate-brainstorm`), NOT architecture/stack (`architect`), NOT module/feature design
(`to-prd`, upstream — don't edit the PRD), NOT writing code or claiming/merging (the
orchestrator, downstream). If the user drifts up into PRD-level design, note it and steer
back — or, if the PRD is genuinely wrong, point them at `to-prd` to amend it (then re-run to
reconcile).

## Fully autonomous, uncertainty-as-tasks
The core de-bias here: a synchronous "quiz the user on granularity" step is
wrong for an AFK-first pipeline. Slice autonomously; derive dependencies from module
interfaces; turn a genuine doubt into a **HITL decision task that blocks its dependents**, not
a blocking prompt. The pipeline never pauses.

## Cheap reconcile
Reads scale with the change: glob (ID-set + coverage signature) → `grep` the `spec-hash` →
body-read ONLY changed slices. A one-module amend touches ~one task whether the feature has 3
or 30. Everything that could drift — readiness, parentage, supersession — is **globbed, not
stored**.

## Anti-patterns
- Blocking to ask the user about granularity or dependencies (externalize as HITL / derive).
- Hashing the generated task body instead of the verbatim PRD projection (non-deterministic →
  everything looks changed).
- Hand-computing `spec-hash` instead of shelling out to `shasum`.
- Mutating a `claimed`/`done` task on change (mint a delta task instead).
- Stamping `superseded-by` onto a frozen task (breaks *never mutate frozen* — derive
  supersession by globbing `amends:`).
- Reusing a freed slice number, or renumbering on reconcile (ID is the dedup key).
- Auto-deleting orphans (surface them; `claimed` orphans → `needs-human/`).
- Storing `id`/parent/status in frontmatter (they're in the filename + directory).
- Giving HITL tasks a separate birth lane, or a stored `blocked/` dir (born in `open/`;
  readiness is derived).
- Editing the PRD, writing code, or moving tasks into `claimed`/`done` (that's the
  orchestrator).
- Publishing to an issue tracker / applying a triage label (the base's tracker mechanic;
  navigate emits files).
</supporting-info>
