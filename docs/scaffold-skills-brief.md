# scaffold-skills — Brief (skill #3)

*Decided 2026-06-01 via `architect` (used meta, on the navigate system itself). Read
alongside `DESIGN.md` and `docs/architect-brief.md`. Glossary terms live in DESIGN.md's
Glossary section.*

## Problem

After `architect` writes the decided architecture / stack / delivery into CLAUDE.md, the
project has **no project-specific `.claude/` config** — the skills, conventions, and
guardrails an agent (orchestrated or interactive) needs to actually build in *this* stack.
`scaffold-skills` is the pipeline stage that **generates that `.claude/` set from the
brief**. It is a de-biased, brief-driven generator: it keeps a library of hand-tuned
template *content* but drops the **interview** (the brief already decided everything) and
the **fixed mapping table** (the source of the taxonomy bias).

**User:** the solo builder + their building agents. **Slot:** pipeline stage #3, after
`architect` (#2), before `to-tasks` (#4). It consumes CLAUDE.md/CONTEXT.md; it writes
`.claude/`.

## Decided shape (the four load-bearing calls)

### A. Generation shape — hybrid **match-or-synthesize**

Read the decided stack/delivery from CLAUDE.md. For each justified concern, **reuse a
harvested template when one fits**, and **synthesize a new artifact from the nearest
exemplar's shape when none does**. Templates are a **quality anchor / pattern library**,
not a catalog.

- *Not* a pure templater — that can't honor "no fixed taxonomy" (a novel stack has no row).
- *Not* a pure synthesizer — that throws away the templates' hand-tuned quality bar.

### B. Output kinds — **skills + rules + safety**

Emit, all **brief-derived** (no fixed catalog of *which*; the artifact *kinds* are Claude
Code platform primitives):

- `skills/` — workflows (e.g. a module-scaffold skill per language, a deploy skill per
  delivery target).
- `rules/` — path-scoped conventions (per language/tool), via `applies_to:` globs.
- `settings.json` — permissions + safety hooks, **scaled to the brief's prod-level**.
- `CLAUDE.md` / `README` — always emitted as the frame.

**Agent-personas (`agents/`) are dropped from the default set.** The orchestrator already
*claims a task and runs an agent in a sandbox*, so a static `backend-dev.md` persona
duplicates that role. Re-addable later if the orchestrator turns out to want a persona to
adopt (roadmap).

### C. Match engine — **self-describing templates + agentic match**

A hardcoded `Go → go-module` mapping table *was* the fixed taxonomy;
it does not come back. Instead:

- Each harvested template carries light frontmatter — `kind: skill|rule`, `applies-to:`
  (tech), `concern:` (what it does) — plus its existing `description:`. (Rules already
  carry `applies_to:` globs; this extends the convention to skill templates.)
- `scaffold-skills` reads the **library metadata + the brief**, and the **agent reasons**
  about fit: reuse-and-fill on a clear match, synthesize-from-nearest-exemplar otherwise.

Selection becomes **data + reasoning**, not code — which is exactly what lets it degrade
gracefully into synthesis for stacks no template anticipated. (Rejected: a catalog/manifest
with match keys — that's the mapping table reincarnated as data, only matching what's
enumerated.)

### D. Re-run model — **idempotent additive reconcile**

`scaffold-skills` is **not once-only**: `architect` re-runs in `change`/`audit` mode and
emits deltas, so re-invocation hits an existing, possibly hand-edited `.claude/`. On
re-run it:

1. Computes the delta between what the **current brief** justifies and what's **already in**
   `.claude/`.
2. **Adds/updates only newly-justified artifacts.**
3. **Never silently overwrites a hand-edited file** — flags conflicts to `needs-human`.
4. **Surfaces orphans** (artifacts whose brief justification was dropped) rather than
   auto-deleting.

Generated-vs-hand-owned is told apart by a **provenance stamp** (a frontmatter hash, like a
familiar "never modify generated files" header): file matches its stamp →
safe to regenerate; diverged → hand-owned, flag don't clobber. This mirrors navigate's own
**infra-first / additive** principle (the running config keeps working across the gap).

### E. Delivery

Ships like every other pipeline skill: a `SKILL.md` plus a bundled, curated `templates/`
directory (each template carrying the phase-C self-describing frontmatter). No separate decision.

## Roadmap (one-liners)

- **Agent-personas, demand-driven** — re-add `agents/` if the orchestrator wants a persona
  to adopt per task class.
- **Provenance/stamp format** — exact hash scheme + where it lives in frontmatter.
- **Synthesis quality bar** — how a synthesized-from-exemplar artifact is validated before
  it's written (lint/shape checks).
- **Template library refresh** — keep the harvested exemplars current as stacks evolve.
- **Orphan policy** — beyond surfacing: archival vs prompt-to-delete.

## Out of scope

- The **interview** — there is none; the brief is the input (that's the whole de-bias).
- Writing application code, IaC, or CI YAML — `scaffold-skills` writes only `.claude/`.
- Deciding the stack/delivery — that's `architect` (#2), upstream.
- Breaking the brief into tasks — that's `to-tasks` (#4), downstream.

## Constraints

- **Brief-driven, never interview.** Read CLAUDE.md/CONTEXT.md; do not ask the user to
  re-state what `architect` decided.
- **No fixed taxonomy** — the emitted *set* is derived from the brief, open-ended; only the
  artifact *kinds* (platform primitives) are fixed.
- **Only `.claude/` content** — never application/infra/CI files.
- **Idempotent + safe** — re-runnable; never clobber hand-edits; flag conflicts/orphans.
- Ship a curated `templates/` library for content; no interview, no mapping table.
