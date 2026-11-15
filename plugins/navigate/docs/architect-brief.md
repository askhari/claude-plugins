# architect — Brief (skill #2)

*Decided 2026-06-01 via `navigate-brainstorm` (used meta, on itself). Read alongside
`DESIGN.md` and `.claude/skills/navigate-brainstorm/`. Glossary terms live in DESIGN.md's
Glossary section.*

## Problem

After the product brief exists (CLAUDE.md + CONTEXT.md), the solo builder and their
agents have no guided, preference-aware way to decide **how** to build or modify the
software — architecture shape, language/tech stack, infrastructure (on-prem vs cloud),
and which cloud provider — so these high-leverage, hard-to-reverse choices get made ad
hoc. `architect` is the interview that decides them.

**User:** the solo builder + their building agents. **Slot:** pipeline stage #2, between
`navigate-brainstorm` (#1) and `scaffold-skills` (#3); its on-prem/cloud outcome can
trigger `carve-repo` (#5).

## Shape: one phased skill, not two

Software-vs-infra is a **blast-radius** axis, which navigate assigns to **repos**
(`carve-repo`), *not* to skills. Skills express **zoom**; architecture + infra are one
zoom level ("how to build"). So `architect` is a **single skill with ordered phases**,
not a software-skill + infra-skill pair. Splitting it would re-introduce the
axis-conflation DESIGN.md identifies as the source of every multi-repo failure mode.

If the infra/cloud phases ever grow heavy, extract them **demand-driven** (mirroring how
`carve-repo` is deferred) — not preemptively.

## Entry modes (one mechanism)

`architect <mode>` — all three share one machinery: survey → phased tree → ADR-on-reversal.

| Mode | Survey? | Phases run | Posture |
|---|---|---|---|
| `new` | no | all (A–E) | decide fresh from the brief |
| `change` | yes | only the phases the change touches | constrained by prior decisions |
| `audit` | yes | all (A–E), evaluate-mode | "is each choice still right at today's scale?" → recommend deltas |

`audit` is just `change` with no predetermined change, walking every phase in evaluate
posture. **Survey** reads an existing codebase's architecture/stack into context before
any recommendation; skipped for `new`.

## MVP phases

```
architect <mode>
  survey            (skip if new)  -> findings into context
  A  architecture   shape / pattern
  B  stack          languages & tech, concrete + scored
  C  infra          on-prem vs cloud
  D  cloud          provider pick, scored on the rubric below
  E  delivery       CI/CD
  -> ADR on any reversal of a prior decision; updates CLAUDE.md
```

Architecture and stack recommendations are **concrete** — architect says "use X, because…"
from its own opinionated knowledge. The **cloud provider is the exception**: architect
carries **no built-in favorite**; it enumerates candidate providers *with the user* during
the interview and lets the weighted criteria pick. Weights come from standing preferences.

## Cloud rubric (phase D)

Three weighted criteria — setup difficulty is **folded into operational cost** as amortized
upfront effort (automation is the lever that converts setup → lower ongoing ops; they
trade off along one total-cost-of-ownership axis, so scoring them separately double-counts
effort):

| Criterion | Weight | Notes |
|---|---|---|
| **Monetary cost** | HIGH | **tiebreaker** when it conflicts with effort |
| **Operational / effort cost** | HIGH | ongoing ops **with setup amortized in** |
| **Vendor lock-in** | MED | |

**Tie rule:** cheaper money wins over lower effort — the solo builder has time but not
budget. **No built-in provider favorite:** these weights are the user's stated priorities,
not a lean toward any provider. Phase D enumerates the candidate providers *with the user*
during the interview and applies the weights to whatever they're considering. Overridable
per project; the per-project interview can re-weight, and durable changes promote to
standing preferences.

## Standing-preferences loop

`architect` **loads** durable defaults before interviewing (skipping settled branches —
preferred languages, cloud posture, lock-in tolerance, rubric weights) and **promotes**
new durable choices after. The prefs *home* (navigate-home vs per-repo) is still open in
DESIGN.md and is not decided by this brief.

## Roadmap (one-liners)

- **Phase F — observability** (logging / metrics / alerts); MVP ships a one-line default.
- **Brownfield survey depth** — richer existing-codebase analysis for `change`/`audit`.
- **Deep delivery patterns** — beyond a basic CI/CD pick in phase E.
- **`carve-repo` handoff** — fire when on-prem/cloud implies a real blast-radius split.
- **Knowledge-base refresh** — keep the stack/provider opinions current.

## Out of scope

- Detailed module/component design (downstream — per-feature / `to-tasks`).
- Writing IaC or provisioning infrastructure (execution — `scaffold-skills` / orchestrator).
- Executing the repo split — `architect` only **makes the topology call**; `carve-repo`
  executes it.

## Constraints

- Must obey the interview method: one question at a time, recommend an answer each step,
  resolve dependencies in order (survey → A → B → C → D → E).
- Writes/updates **CLAUDE.md** (brief) and emits **ADRs** under `docs/adr/` on reversals;
  glossary terms go to **CONTEXT.md**.
