# navigate — Design Record

*Resume of the design session (2026-05-31). Read this alongside `.claude/skills/navigate-brainstorm/`
when building the remaining skills (`architect`, `scaffold-skills`, `to-prd`, `to-tasks`, `carve-repo`).*

## What navigate is

A system that takes a fuzzy idea to running, agent-built software. It works through a
**pipeline of scoped interview + templating skills** operating inside a **single repo**,
producing the artifacts (brief, glossary, ADRs, tasks) that autonomous agents build from.

## Core model: skills express zoom, repos express blast-radius

We separated two axes we had been conflating:

- **Zoom** (what → architecture → tech → tasks) is expressed by **skills** in a pipeline.
- **Blast-radius** (infra vs app — separated for creds, lifecycle, permissions) is
  expressed by **repos**.

So: **start in one repo.** Topology is not an assumption but a **decision the `architect`
skill makes**, executed by a deferred **`carve-repo`** skill only when a real blast-radius
boundary exists. Every serious failure mode we found (cross-repo straddles, decision drift,
cross-repo claiming, isolation/projection) was an artifact of the multi-repo assumption;
collapsing to one repo deletes them all. Keeping artifacts self-contained + git-native
keeps a future split cheap.

## File & naming conventions

```
<project repo>
├── CLAUDE.md       ← the brief (distilled, ambient — loads every turn); references CONTEXT.md
├── CONTEXT.md      ← glossary / terms
├── docs/adr/0001-*.md
├── docs/prd/0007-slug.md   ← per-feature PRD (durable parent); docs/prd/ = the feature ledger
├── .claude/        ← project-specific skills/agents/rules (from scaffold-skills)
└── tasks/{open,claimed,done,needs-human}/0007-03-slug.md   ← git-native; status = directory; ID = PRD-slice (parent encoded in filename)
```

Key point: **CLAUDE.md = brief, CONTEXT.md = glossary.** This makes the brief ambient
(auto-loaded for every agent) AND uses the standard glossary convention, so the
tier-1 interview needs **no special variant**.

## The skill pipeline

| # | Skill | Does | Writes | Status |
|---|---|---|---|---|
| 1 | **navigate-brainstorm** | product tier-0: what/who/why, MVP, roadmap | CLAUDE.md, CONTEXT.md | **v0 written** |
| 2 | **architect** | coarse architecture, stack, delivery (+observability roadmap); 3 entry modes (new/change/audit); decides if/when topology splits | updates CLAUDE.md | **brief decided** ([docs/architect-brief.md](docs/architect-brief.md)) |
| 3 | **scaffold-skills** | generate the project-specific `.claude/` skill set from the stack | `.claude/` | **brief decided** ([docs/scaffold-skills-brief.md](docs/scaffold-skills-brief.md)) |
| 4 | **to-prd** | **feature zoom**: zoom into ONE roadmap item → modules, user stories, testing decisions | `docs/prd/NNNN-slug.md` | **v1 written** ([brief](docs/to-prd-brief.md), `.claude/skills/to-prd/`) |
| 5 | **to-tasks** | slice one PRD into AFK/HITL vertical-slice tasks (tracer bullets) | `tasks/` | **v1 written** ([brief](docs/to-tasks-brief.md), `.claude/skills/to-tasks/`) |
| 6 | **carve-repo** *(deferred, demand-driven)* | extract a self-contained subtree into a new blast-radius-isolated repo | new repo + manifest | todo |

Steps 1–3 run ~once per project. **`to-prd → to-tasks` is the repeating inner loop**:
pick a roadmap item, PRD it, slice it — realizing the brief one feature at a time. This
completes the `what → architecture → tech → feature` fractal (`to-prd` *is* the feature zoom
the glossary already names).

The orchestrator (the user's own Rust/Go binary) drains `tasks/` — no manifest or
isolation machinery until `carve-repo` fires.

## Glossary (the shared vocabulary)

- **navigate** — the system: fuzzy idea → running, agent-built software.
- **interview** — relentless one-question-at-a-time method that walks a decision tree;
  recommends an answer each step.
- **fractal brainstorm** — the same interview at successive zooms (what → architecture →
  tech → feature), realized as a **skill pipeline**, not a repo hierarchy.
- **brief** — the decided product/architecture/tech context. Lives in **CLAUDE.md**.
- **brief projection** — a **complete, self-contained copy** of everything a repo needs,
  baked in at carve time (**static linking** — not a live reference).
- **standing preferences** — the user's compounding defaults; loaded before an interview,
  updated after, so each project asks fewer questions. (Home location: open.)
- **topology** — the set of repos and their boundaries; a *decision*, not an assumption.
- **blast radius** — the risk a change can cause; the rationale for repo boundaries
  (infra high, app low).
- **infra-first** — any change needing infrastructure does the infra part first and
  **additively** (the running app keeps working in the gap); cleanup is a later
  infra-first change; truly-atomic cross-repo changes are a HITL escape hatch.
  *(Absorbs the old "expand-contract" term.)*
- **straddle** — a change that crosses a repo boundary.
- **task queue** — per-repo, git-native dirs `open/ claimed/ done/ needs-human/`;
  status = which directory.
- **PRD (feature zoom)** — the per-feature spec produced by `to-prd` for ONE roadmap item:
  modules (deep-module decomposition), user stories, testing decisions. Finer than the
  brief (whole-project), coarser than tasks (layer slices). Durable & reviewable — it does
  NOT move through the task dirs. Lives in `docs/prd/NNNN-slug.md`.
- **feature ledger** — `docs/prd/` as a whole: the chronological, numbered record of every
  feature ever specced. The artifact that makes a long-lived project traceable.
- **derived parentage** — tasks link to their PRD by **stable number only** (child → parent,
  e.g. task ID `0007-03` ⇒ belongs to PRD `0007`). Parent → children is **never stored** —
  it's derived on demand (`ls tasks/*/0007-*`), because tasks move between status dirs and a
  stored child-list would drift. "Feature done?" = all of `0007-*` are in `done/` (derived,
  not tracked). Same anti-manifest reasoning as `carve-repo` deferral.
- **to-prd shape (thin hybrid)** — `to-prd` **synthesizes** the PRD from the rich upstream
  brief, then runs a single **interview seam** — the deep-module decomposition + which
  modules get tested — and iterates only there. Not a pure synthesizer (modules
  too load-bearing to guess), not a full feature-zoom interview (re-asks what the brief
  settled). Resolves DESIGN.md's open architect-altitude question. See [docs/to-prd-brief.md](docs/to-prd-brief.md).
- **module seam** — the one interactive checkpoint in `to-prd`: present the deep-module
  decomposition (responsibility · interface sketch · tested?) and iterate to approval. The
  PRD's `Modules` section is **first-class** (promoted out of a generic "Implementation
  Decisions" grab-bag) because it's both the seam and `to-tasks`' slice source.
- **amend-in-place (PRD)** — re-running `to-prd` on a living feature edits the **same**
  `NNNN-slug.md`; **git is the history**; the number is stable forever. Required by
  *derived parentage* (`0007-*` tasks must keep resolving to PRD `0007`). Rejected
  alternatives: append-only/supersede (orphans tasks), create-only (no re-slice path).
- **task reconcile** — `to-tasks`' re-run is a **reconcile, not a regenerate**: the `tasks/`
  expression of *idempotent additive reconcile*. Status-aware (the directory says what you
  may do), **task ID = dedup key** (same slice → same `0007-NN`, new numbers `max+1`, never
  reused), **never mutates `claimed`/`done`** (emits a *delta task* instead), **surfaces
  orphans** (no auto-delete), **no manifest** (glob + reasoning). Always-on; no amend mode.
- **delta task** — a new follow-up task minted when a *changed* slice's existing task is
  `claimed`/`done` — because in-flight/shipped work is never mutated (infra-first/additive).
  Identity (decided in `to-tasks`): a fresh `max+1` task in `open/`, current coverage +
  `spec-hash`, with an **`amends: NNNN-MM`** back-pointer to the frozen task it revises.
- **derived supersession** — a frozen (`claimed`/`done`) task is "superseded" iff some delta
  task carries `amends: <its-id>`; reconcile **globs for `amends:`** rather than stamping a
  `superseded-by` on the original (which would mutate a frozen task). Skipping already-amended
  frozen tasks stops the infinite-delta chain. Same anti-stored-state reasoning as *derived
  parentage*.
- **coverage signature** — a task's `(covers-stories, modules)` frontmatter pair; the
  structural **slice-identity key** `to-tasks` reconcile matches on (glob + frontmatter scan,
  no body reads). A vertical slice's story-set usually discriminates it even when module-sets
  overlap. *Signature-first, fuzzy fallback*: residual unmatched tasks/slices (coverage
  *drift* — rename or added story) get a bounded reasoning pass (overlap + slug + one body
  read) so drift reads as *changed*, not orphan+new — the "matched by reasoning on content"
  the `to-tasks` contract names.
- **spec-hash stamp** — a task's frontmatter hash of the slice spec it was generated from
  (the `tasks/` instance of the **provenance stamp**). Lets `task reconcile` tell changed
  from unchanged with a one-line header scan, bounding body reads to *only changed slices* —
  reconcile cost scales with the change, not the feature. Contract on `to-tasks`: it must
  stamp every task. **Hashes the PRD *source*, not the generated task prose** (LLM prose
  varies run-to-run → would mark everything changed): a **per-slice projection** of the
  covered user-story lines + module blocks, pulled verbatim by the *coverage signature*,
  canonicalized and **`shasum`'d by a shell tool** (first 12 hex) — never hand-computed.
- **AFK / HITL** — a task an agent can do unattended / a task needing the user.
- **scope: portfolio / scope: repo** — tag on a HITL item; portfolio-scoped ones bubble up.
- **entry mode** — how `architect` is invoked: `new` (greenfield), `change` (modify
  existing software), `audit` (evaluate whether the existing architecture needs changes).
  All share one mechanism; `audit` = `change` with no predetermined change.
- **survey** — `architect`'s phase-0 read of an existing codebase's architecture/stack into
  context before recommending; runs for `change`/`audit`, skipped for `new`.
- **rubric (cloud)** — `architect` phase-D scoring on monetary cost (HIGH, tiebreaker),
  operational/effort cost with setup amortized in (HIGH), and vendor lock-in (MED).
- **effort (TCO)** — total cost of ownership over the project horizon: ongoing operational
  cost **with upfront setup amortized in**. Setup and ops trade off along this one axis
  (automation converts setup → lower ops), so they are scored together, not separately.
- **ADR-on-reversal** — `architect` emits an ADR only when it reverses a previously decided
  architecture/stack/infra choice (a hard-to-reverse, surprising-in-hindsight trade-off).
- **down-and-up (bubble-up)** — a repo-level discovery that should amend a higher-level
  decision, surfaced as a `scope: portfolio` HITL item.
- **orchestrator** — the user's static binary (Rust or Go) that drains AFK tasks:
  claim → run agent in sandbox → merge → done.
- **claim** — the orchestrator reserving a task before dispatch — the single
  serialization point (the `git mv` is just bookkeeping).
- **manifest** — only when multi-repo: the parent's record of repos (build order, status,
  pinned SHA = soft lockfile, bubble-up list). **No git submodules.**
- **match-or-synthesize** — `scaffold-skills`' generation shape: reuse a harvested template
  when one fits the brief, synthesize a new artifact from the nearest exemplar's shape when
  none does. Hybrid of templating and generation.
- **quality anchor / exemplar** — a harvested template used as a *pattern + quality bar*
  (a few-shot reference), not as a fixed catalog entry. The core de-bias of `scaffold-skills`.
- **self-describing template** — a template carrying light frontmatter (`kind`,
  `applies-to`, `concern`) the agent *reasons over* to decide fit — replacing
  a hardcoded mapping table (which *was* the fixed-taxonomy bias).
- **provenance stamp** — a frontmatter hash marking a `.claude/` file as `scaffold-skills`-
  generated, so a re-run can tell generated (safe to regenerate) from hand-owned (flag,
  don't clobber).
- **idempotent additive reconcile** — `scaffold-skills`' re-run model: diff brief↔`.claude/`,
  add/update only newly-justified artifacts, flag conflicts to `needs-human`, surface
  orphans rather than auto-delete, never overwrite hand-edits. The `.claude/`-generation
  expression of **infra-first / additive**.

## Resolved decisions

- Single-repo-first; topology demoted to an `architect` decision; `carve-repo` deferred.
- **No git submodules.** If/when multi-repo, link via a manifest-lockfile (pinned SHA);
  repos are fully isolated (static-linked at carve).
- **infra-first** absorbs expand-contract (see glossary).
- Tasks are git-native files; AFK vs HITL; status = directory; HITL `scope: portfolio`
  bubbles up.
- Orchestrator = the user's own static binary — not an off-the-shelf tool, and separate
  from the project repo. Claim is the single serialization point.
- CLAUDE.md = brief, CONTEXT.md = glossary → the tier-1 interview needs **no special variant**.
- `scaffold-skills` = de-biased, **brief-driven** generator
  (no interview, no fixed taxonomy; ships a curated template library). **Brief decided**
  ([docs/scaffold-skills-brief.md](docs/scaffold-skills-brief.md)): (A) **match-or-synthesize**
  shape; (B) emits **skills + rules + safety** (drop `agents/` personas — the orchestrator
  is the agent; `CLAUDE.md`/`README` as frame); (C) **self-describing templates + agentic
  match** replaces the mapping table; (D) **idempotent additive reconcile** with a
  **provenance stamp** (the `.claude/` expression of infra-first/additive).
- The interview method is named **interview**; skill #1 is **navigate-brainstorm**.
- **`to-tasks` splits into `to-prd` + `to-tasks`** (this session). `to-prd` is the missing
  **feature zoom** (completes `what → arch → tech → feature`); the two form the **repeating
  inner loop** (per-feature), while steps 1–3 run once per project. Rationale: separate
  *zoom* (PRD decomposes one feature; tasks slice layers), separate *lifecycle* (PRD durable
  & reviewable; tasks disposable), and *longevity* (per-feature PRDs accrete into a
  traceable ledger; a whole-project PRD goes stale at feature #2). PRDs live in
  `docs/prd/NNNN-slug.md`; tasks use **hierarchical IDs** `PRD-slice` (`0007-03-slug.md`)
  so the parent link is visible in the filename and children are enumerable by glob —
  **child → parent stored, parent → children derived** (see *derived parentage*).
- **`to-prd` brief decided** ([docs/to-prd-brief.md](docs/to-prd-brief.md), via `architect`
  meta). Resolves the open interview-vs-synthesize call: **thin hybrid** — synthesize from
  the rich brief, interview only at the **module seam** (see glossary). Template promotes
  `Modules` to first-class. Lifecycle = **amend-in-place** (stable
  number, git = history), which forces `to-tasks`' re-run to be a **task reconcile**
  (idempotent additive, status-aware, ID = dedup key, delta tasks for `claimed`/`done`),
  made cheap by a **spec-hash stamp** so reconcile reads scale with the change, not the
  feature. The reconcile truth-table + stamp are a **contract on `to-tasks`**, recorded for
  its brief.
- **`to-tasks` brief decided + built** ([docs/to-tasks-brief.md](docs/to-tasks-brief.md),
  `.claude/skills/to-tasks/`, via `architect` meta). Six load-bearing calls: (A) **fully
  autonomous** — no blocking interview; dependencies *derived* from module interfaces, and any
  unresolvable doubt becomes a **HITL decision task** that blocks its dependent AFK slices
  (the slicing quiz made async). (B) **All tasks born in `open/`**; `type:AFK|HITL` is a
  label, `blocked-by` the universal mechanism, **readiness derived** (ready iff blockers in
  `done/`), `needs-human/` = escalation only. (C) **Structured frontmatter** (`type`,
  `covers-stories`, `modules`, `blocked-by`, `tests`, `scope`-HITL, `spec-hash`) — but nothing
  derivable from filename/dir (`id`/parent/status). (D) **`spec-hash` over the verbatim PRD
  projection** (covered stories + module blocks), per-slice, via `shasum`. (E) **Slice
  identity = coverage signature, fuzzy fallback** for drift. (F) **Delta-task identity** =
  fresh `max+1` + `amends:` + **derived supersession**. The reconcile truth-table (formerly a
  contract recorded in `to-prd`) is now **owned by `to-tasks`**.
- **architect is one phased skill, not two.** Software-vs-infra is a *blast-radius* axis
  (→ repos/`carve-repo`), not a *zoom* axis (→ skills); splitting would re-conflate the two.
  One skill, ordered phases: survey → A arch → B stack → C infra (on-prem/cloud) → D cloud
  → E delivery. Observability deferred to roadmap (phase F).
- **architect has 3 entry modes** sharing one mechanism: `new` (no survey, all phases),
  `change` (survey, only touched phases), `audit` (survey, all phases in evaluate-mode).
  Emits an ADR on any reversal of a prior decision.
- **architect recommends concretely** for architecture and stack (not a neutral
  facilitator). The **cloud provider is the exception** — no built-in favorite; phase D
  enumerates candidates *with the user* and applies a **3-criterion weighted rubric**:
  monetary cost (HIGH, tiebreaker) · operational/effort cost with setup amortized in (HIGH)
  · vendor lock-in (MED). Tie rule: cheaper money wins over lower effort. The weights are
  the user's priorities (not a provider lean); overridable per project, from standing prefs.

## Open items

- **Standing preferences home** — a navigate home above projects vs copied into each
  repo. Unresolved; affects navigate-brainstorm step 4 **and** architect (which loads/
  promotes durable stack/cloud/lock-in defaults + rubric weights).
- **Manifest format** (yaml/toml/json) — only matters once `carve-repo` exists.
- **Orchestrator** — language (Rust vs Go) and claim/merge mechanics. Now the **last
  unbuilt pipeline piece**: it drains `open/ ∧ type:AFK ∧ ready` (readiness derived from
  `blocked-by` + `done/`), claims (the single serialization point), runs the agent in a
  sandbox, merges → `done/`; the human works `open/ ∧ type:HITL ∧ ready` + `needs-human/`.
  `to-tasks` defines the file contract it consumes (see [docs/to-tasks-brief.md](docs/to-tasks-brief.md)).
- **`carve-repo`** *(deferred, demand-driven)* — still todo; fires only on a real
  blast-radius split.
