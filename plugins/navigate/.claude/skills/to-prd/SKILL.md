---
name: to-prd
description: Feature zoom — take ONE roadmap item to a durable, reviewable PRD (modules, user stories, testing decisions) and write it to the `docs/prd/` feature ledger. Synthesizes from the rich brief, interviews only at the module decomposition. Fourth skill in the navigate pipeline; follows `scaffold-skills`, hands off to `to-tasks`. Use to spec a feature, or on "PRD this", "to-prd", "zoom into <roadmap item>".
---

<what-to-do>
Take ONE roadmap item to a PRD. You are the **feature zoom** — finer than the brief
(whole-project), coarser than tasks (layer slices). The brief (CLAUDE.md + CONTEXT.md) is
already rich, so do **not** run a full interview: **synthesize** the PRD from what's
already decided. The one exception is the **module seam** — the deep-module decomposition,
which the brief does not contain and which is the least-recoverable call in a PRD. Present
that, and only that, for the user to shape.

Stay at FEATURE altitude: modules, user stories, testing decisions. Layer-by-layer task
slicing is downstream (`to-tasks`); writing code is execution (the orchestrator). One PRD
per roadmap item. Use the project's glossary vocabulary; respect ADRs in the touched area.
Coin/record any new term in CONTEXT.md.
</what-to-do>

<inputs>
Read before drafting anything:

- **The roadmap item** — passed as the argument (a roadmap line from CLAUDE.md, or a short
  phrase). This is the ONE feature you zoom into.
- **CLAUDE.md** — the decided brief: problem, MVP, roadmap, architecture, stack, delivery.
  This grounds the synthesis; don't re-ask what it settles.
- **CONTEXT.md** — glossary; use its vocabulary throughout the PRD.
- **`docs/adr/`** — respect existing decisions in the area you're touching.
- **The codebase** — explore the current state of the modules you'll build or modify.
- **`docs/prd/`** — the existing feature ledger: to pick the next `NNNN`, and to detect
  whether this feature already has a PRD (→ amend, see `<lifecycle>`).

If the roadmap item is ambiguous or absent, ask which feature to zoom into — that one
question only. Everything else comes from the brief.
</inputs>

<process>
1. **Synthesize the draft silently.** From the brief + roadmap item + codebase, draft the
   PRD's settled sections — Problem, Solution, User Stories (a long, extensive numbered
   list), Out of Scope. These the brief already grounds; don't interview them.

2. **The module seam — the one interactive checkpoint.** Sketch the **deep-module**
   decomposition: actively look to extract modules that encapsulate a lot of functionality
   behind a simple, testable interface that rarely changes (deep, not shallow). Present
   them and ask the user:
   - Do these modules match your expectations? (merge / split / rename?)
   - Which modules do you want tests written for?

   Iterate **only at this seam** until the user approves. This is the whole interview.

3. **Write the PRD** to `docs/prd/NNNN-slug.md` using the template below. Then hand off to
   `to-tasks`. No issue tracker, no triage label — the file in the ledger *is* the publish.
</process>

<prd-template>
Promote `Modules` to a first-class section — it's the seam AND `to-tasks`' slice source.

```
## Problem
The problem the user faces, from the user's perspective.

## Solution
The solution, from the user's perspective.

## User Stories
A LONG, extensive numbered list. Each: "As an <actor>, I want <feature>, so that <benefit>."
Cover all aspects of the feature.

## Modules
The deep-module decomposition (the approved seam). For each module:
- **name** — responsibility (the functionality it encapsulates)
- **interface sketch** — the simple, stable surface (not code; the shape)
- **tested?** — yes/no + one-line why (drives `to-tasks`' testing slices)

## Other Decisions
Implementation decisions that aren't module-shaped: schema changes, API contracts,
architectural clarifications, specific interactions. NO file paths or code snippets —
they go stale fast. Exception: a prototype-derived snippet that encodes a decision more
precisely than prose (state machine, reducer, schema, type shape) — inline the
decision-rich bits only, noting it came from a prototype.

## Testing Decisions
What makes a good test here (test external behavior, not implementation details); which
modules are tested (from the seam); prior art (similar tests in the codebase).

## Out of Scope
What this PRD deliberately does not cover.

## Further Notes
Anything else worth recording.
```
</prd-template>

<lifecycle>
**Numbering.** `NNNN` = the next free zero-padded 4-digit integer in `docs/prd/` — the same
convention as `docs/adr/0001-` and task IDs (`0007`). The slug comes from the roadmap item.

**Amend-in-place.** If the feature already has a PRD, re-running edits the **same**
`NNNN-slug.md` — never a new number. **Git is the history.** Re-synthesize and re-run the
module seam so the decomposition stays current. The number is **stable forever**: tasks
point at their PRD by number (`tasks/*/0007-*` ⇒ PRD `0007`, *derived parentage*), so
renumbering a living feature would orphan its tasks. Do not append-or-supersede; do not
mint a second PRD for the same feature.

**Why the number must stay put (the `to-tasks` contract).** `to-tasks`' re-run is a
*reconcile*, not a regenerate — it finds this feature's existing tasks by globbing
`tasks/*/NNNN-*` and reconciles them against the amended PRD (new slices created,
in-flight/done work left untouched via delta tasks, orphans surfaced). That whole mechanism
keys off the **stable `NNNN`**. Your only obligation to it is: **keep the number across
amends.** (The reconcile + `spec-hash` stamping live in `to-tasks`, not here.)
</lifecycle>

<outputs>
- Writes **`docs/prd/NNNN-slug.md`** — the durable, reviewable PRD. It does **NOT** move
  through the task dirs; it's the parent, not a task.
- Records any new term in **CONTEXT.md**.
- Reports a tight summary: the PRD path, the approved modules (and which are tested), and —
  on an amend — what changed since the prior version.
- **No ADR** — a PRD is feature design downstream of architecture decisions; ADRs belong to
  `architect`.

Hand off to **`to-tasks`** once the PRD is written.
</outputs>

<supporting-info>
## Altitude
Feature design — modules, stories, testing. NOT product scope (that's `navigate-brainstorm`,
upstream), NOT architecture/stack (that's `architect`, upstream), NOT layer-by-layer task
slices (that's `to-tasks`, downstream), NOT code (the orchestrator). If the user drifts up
into product scope or down into task slicing, note it and steer back.

## Synthesize-first, interview-once
The core de-bias here: the rich upstream brief means a full feature-zoom
interview would re-ask what's already settled — so synthesize. But pure synthesis would
*guess* the deep-module decomposition, the load-bearing part. So keep exactly one seam.

## Anti-patterns
- Interviewing the user for problem/solution/stories the brief already grounds.
- Guessing the module decomposition instead of confirming it at the seam.
- Minting a new `NNNN` (or a superseding PRD) for a feature that already has one — breaks
  derived parentage and orphans the feature's tasks. Amend in place.
- Burying modules in a prose bullet list — keep the `Modules` section first-class.
- Publishing to an issue tracker / applying a triage label (that was the base's tracker
  mechanic; navigate emits a file).
- Including file paths or code snippets (except a decision-encoding prototype snippet).
- Moving the PRD into a task dir, or slicing it into tasks yourself — that's `to-tasks`.
</supporting-info>
