---
name: navigate-brainstorm
description: Product tier-0 interview — take a fuzzy idea to a decided product brief (problem, MVP, roadmap) and seed CLAUDE.md + CONTEXT.md. First skill in the navigate pipeline; hands architecture and tech off to `architect`. Use at the start of a new project, or on "navigate" / "brainstorm what to build".
---

<what-to-do>
Run a relentless product interview: one question at a time, recommend an answer each
step, wait for feedback before the next. Walk the tree, resolving dependencies in order.

Load standing preferences first (if present) and SKIP every branch they settle.
Explore any referenced material instead of asking.

Stay at PRODUCT altitude — what to build and for whom. Architecture, stack, and repo
topology belong to the next skill (`architect`); if the user drifts there, note it and
steer back. Terms are defined in CONTEXT.md — coin and record new ones as they surface.
</what-to-do>

<decision-tree>
1. Problem & user — force a one-sentence problem statement in the user's words: who,
   what pain, why now.
2. MVP boundary — the smallest slice that is demoable and worth shipping. Name
   everything else explicitly and push it to the roadmap.
3. Out-of-scope & constraints — what this deliberately is NOT, plus hard constraints
   (deadline, budget, compliance, "never do X").
4. Standing-prefs reconciliation — promote durable product defaults; record
   project-specific deviations as local notes.
</decision-tree>

<outputs>
Write lazily — only once you have something real.
- CLAUDE.md — the distilled brief: problem, MVP, roadmap (one-liners), constraints.
  Keep it ambient-light (it loads every turn); link deeper detail into docs/.
  Reference CONTEXT.md for terms.
- CONTEXT.md — the glossary: every domain term coined during the interview, defined once.
Hand off to `architect` once the product brief is decided.
</outputs>

<supporting-info>
## Altitude
Product only. Boxes-and-arrows, stacks, repos, and tasks are downstream skills. The
navigate fractal is realized as a skill pipeline, not a repo hierarchy.

## Standing preferences
If a navigate home with PREFERENCES.md exists, load it first, skip settled branches,
and update it at the end. (Its home — navigate-wide above projects, or a repo-local file —
is configurable; if none is set, skip this step.)

## ADRs
Offer one only when a decision is hard to reverse, surprising without context, and the
result of a real trade-off. Otherwise it just lives in CLAUDE.md.
</supporting-info>
