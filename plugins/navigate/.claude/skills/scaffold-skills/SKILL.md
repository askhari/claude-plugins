---
name: scaffold-skills
description: Brief-driven generator — turn a decided architecture/stack/delivery (CLAUDE.md + CONTEXT.md) into a project-specific `.claude/` set (skills, rules, safety hooks) by matching or synthesizing from a bundled, curated template library. Third skill in the navigate pipeline; follows `architect`, hands off to `to-tasks`. Interview-free and brief-driven — no fixed taxonomy. Use after the architecture is decided, or on "scaffold the skills", "generate .claude", "scaffold-skills".
---

<what-to-do>
Generate this project's `.claude/` config **from the decided brief** — never from an
interview. By the time you run, `navigate-brainstorm` and `architect` have written the
stack and delivery into CLAUDE.md; read it and act. Ask the user nothing that the brief
already answers.

Work by **match-or-synthesize**: for each concern the brief justifies, reuse a harvested
template when one fits, and synthesize a new artifact from the nearest exemplar's shape
when none does. Templates are a **quality anchor**, not a catalog — there is **no fixed
taxonomy**; the emitted *set* is derived from the brief, open-ended.

Emit only `.claude/` content (skills, rules, safety hooks, the CLAUDE.md/README frame).
Never write application code, IaC, or CI YAML — that's the orchestrator's and the user's
job. Stay at this altitude: you generate *config*, you don't build the software.

Be **idempotent**: you will be re-run after `architect` emits a delta. Reconcile against
the existing `.claude/`; never clobber a hand-edit. Coin/record any new term in CONTEXT.md.
</what-to-do>

<inputs>
Read before generating anything:

- **CLAUDE.md** — the decided brief. Extract: architecture shape, language(s) +
  framework(s), datastore(s), infra/cloud target, delivery (CI/CD), and **prod-level**
  (the safety dial).
- **CONTEXT.md** — glossary + survey findings; for `change`/`audit` projects this is where
  the existing stack lives.
- **The bundled template library** (`templates/` — a curated, shape-diverse exemplar set:
  skills for compiled/interpreted/IaC stacks, rules for several languages + frontend + IaC,
  plus the `CLAUDE.md`/`README.md`/`settings.json` frame). Each template **self-describes**
  via its frontmatter — skills by `name` + `description`, rules by their `applies_to:` path
  globs — and its body. You match against *that metadata by reasoning*, not a hardcoded
  table. The library is **exemplars, not a catalog**: when nothing fits, synthesize from the
  nearest one's shape.
- **The existing `.claude/`** (if any) — for the reconcile pass.

If CLAUDE.md has no decided stack/delivery yet, stop and send the user back to `architect`
— do not interview them yourself.
</inputs>

<engine>
The **match-or-synthesize** loop. For each concern the brief justifies (a language wants a
module-scaffold skill + a conventions rule; a delivery target wants a deploy skill; the
prod-level wants safety hooks):

1. **Match.** Scan the template library's frontmatter for one whose `kind`/`applies-to`/
   `concern` fits this brief fact. Judge fit by **reasoning**, not string-equality — a
   `go-module` template fits "Go service" even though the words differ.
2. **Reuse-and-fill** on a clear match: instantiate the template, filling its specifics
   from the brief (module path, framework, CI platform, prod rules). Keep the template's
   opinionated content and its `<!-- WHY: … -->` rationale comments.
3. **Synthesize** when nothing fits: take the **nearest exemplar** of the same `kind` as a
   shape reference and write a new artifact for this tech/workflow — same structure, same
   quality bar, same self-describing frontmatter. A Deno+Fly.io stack with no template
   still gets a coherent skill, modelled on the closest one.
4. **Scale safety to prod-level.** `settings.json` hooks tighten as prod-level rises
   (block more, confirm more); a dev-only project gets a lighter set.

Selection is **data + reasoning** (the de-bias), so the library can be *incomplete* and the
skill still produces something sensible. Do **not** reintroduce a `tech → file` lookup table.
</engine>

<what-to-emit>
All **brief-derived** — the *kinds* are Claude Code primitives; *which* ones is open-ended:

- `skills/` — workflows (per-language module scaffold, per-target deploy, test/lint loops).
- `rules/` — path-scoped conventions via `applies_to:` globs (one per language/tool the
  brief names).
- `settings.json` — permissions + safety hooks, **scaled to prod-level**.
- `CLAUDE.md` / `README.md` — always emitted as the frame (the README records what was
  generated and from which brief facts, so a re-run is auditable).

**Do NOT emit `agents/` personas by default.** The orchestrator already claims a task and
runs an agent in a sandbox — a static `backend-dev.md` duplicates that role. (Re-add only
if a project's brief calls for a persona to adopt.)
</what-to-emit>

<reconcile>
**Idempotent additive reconcile** — you are not a once-only generator.

1. **Stamp** every file you generate with a **provenance** marker — `generated-by:
   scaffold-skills` plus a content hash in frontmatter (for non-frontmatter files like
   `settings.json`, a header comment carrying the same). This is how a re-run tells your
   output from a hand-edit.
2. On **re-run**, diff what the *current* brief justifies against what's *already in*
   `.claude/`:
   - **Newly justified** → generate it (match-or-synthesize).
   - **Still justified, file matches its stamp** → safe to regenerate in place.
   - **Still justified, file diverged from its stamp** → **hand-owned**: do not overwrite;
     surface it as a `needs-human` conflict with the diff.
   - **No longer justified** (orphan — the brief dropped the tech) → **surface it**, don't
     auto-delete.
3. Never clobber a hand-edit silently. The running config keeps working across the gap
   (the `.claude/` expression of **infra-first / additive**).
</reconcile>

<outputs>
- Writes the project's **`.claude/`** (skills, rules, settings, CLAUDE.md, README).
- Records new terms / generation notes in **CONTEXT.md**.
- Reports a tight summary: what was generated, what was **synthesized** (no template
  existed), what was skipped and why, and any `needs-human` conflicts / orphans.
- **Does not** write an ADR — generation is downstream of decisions; ADRs belong to
  `architect`.

Hand off to **`to-tasks`** once `.claude/` is generated.
</outputs>

<supporting-info>
## Altitude
You generate **config**, not software. Module/component design is downstream
(per-feature / `to-tasks`); writing IaC, CI YAML, or app code is execution (orchestrator /
user). If the brief is under-decided, send the user back up to `architect` — don't fill the
gap with an interview.

## De-bias
Two things are deliberately gone versus an interview-first generator: the **interview**
(replaced by reading the brief) and the **fixed `tech → file` mapping table** (replaced by
self-describing templates + agentic match). The bundled `templates/` are a vendored, curated
exemplar library — content/quality anchors only, never a taxonomy.

## Anti-patterns
- Interviewing the user for stack/delivery the brief already decided.
- A hardcoded `tech → file` table, or refusing to emit for a stack no template covers
  (synthesize instead).
- Emitting `agents/` personas by default, or any non-`.claude/` file.
- Overwriting a diverged (hand-edited) file, or auto-deleting an orphan.
- Generated files without a provenance stamp (breaks the next reconcile).
</supporting-info>
