---
name: architect
description: Architecture & tech tier-1 interview — take a decided product brief to a decided build plan (architecture shape, stack/languages, infra on-prem-vs-cloud, cloud provider, CI/CD) and update CLAUDE.md. Second skill in the navigate pipeline; follows `navigate-brainstorm`, hands off to `scaffold-skills`, and makes the topology call that may trigger `carve-repo`. Use after the product brief exists, or on "architect", "decide the stack/architecture", or to audit whether an existing architecture still fits.
---

<what-to-do>
Run a relentless architecture interview: one question at a time, recommend an answer each
step, wait for feedback before the next. Walk the phases in order — dependencies flow
downward (architecture constrains stack, stack + infra constrain cloud, cloud constrains
delivery).

Load standing preferences first (if present) and SKIP every branch they settle.
Explore the brief (CLAUDE.md + CONTEXT.md) and any referenced material instead of asking
what's already decided.

Recommend CONCRETELY for architecture and stack — name specifics from your own opinionated
knowledge; you are not a neutral facilitator there. For the CLOUD provider, do NOT carry a
built-in favorite: explore the candidate options together with the user during the
interview and let the weighted criteria pick. Stay at ARCHITECTURE altitude: coarse shape
and load-bearing choices, NOT detailed module design, NOT writing IaC, NOT executing a repo
split. If the user drifts down into per-feature design or up into product scope, note it
and steer back. Coin and record new terms in CONTEXT.md.
</what-to-do>

<entry-mode>
Establish the mode first — ask only if it isn't obvious from context:

- `new`   — greenfield. No survey. Run all phases (A–E) fresh from the brief.
- `change`— modifying existing software with a change in hand. Run survey, then only
            the phases the change touches, constrained by prior decisions in CLAUDE.md.
- `audit` — existing software, no predetermined change. Run survey, then walk all
            phases in evaluate posture ("is each choice still right at today's
            scale/brief?") and recommend deltas.

`audit` is `change` with no predetermined change. For `change`/`audit`, run survey before
any recommendation.
</entry-mode>

<survey>
(change / audit only) Read the existing codebase into context before recommending:
current architecture shape, languages/frameworks/datastores, infra & deploy target,
CI/CD. Write findings to CONTEXT.md so later phases — and downstream skills — can rely on
them. Treat surveyed choices as the prior decisions each phase is measured against; only
recommend a change when a phase's evaluation justifies it.
</survey>

<decision-tree>
Walk in order. Skip any phase/branch settled by standing preferences or the survey; for
`change`, run only the phases the change touches.

A. Architecture shape — the coarse pattern (monolith / modular monolith / services /
   event-driven / job+worker / static+functions …). Pick the simplest shape the brief's
   MVP and scale justify; name what it is deliberately NOT.
B. Stack & languages — concrete language(s), framework(s), datastore(s). Recommend
   specifics with a one-line why; weigh against standing-pref defaults and team skill.
C. Infra: on-prem vs cloud — decide the hosting model. If this implies a real
   blast-radius boundary (infra lifecycle/creds separate from app), flag a topology split
   for `carve-repo` — don't execute it here.
D. Cloud provider — only if C chose cloud. Enumerate candidate providers WITH the user,
   score each together on the criteria below, and let the score pick. No built-in favorite.
E. Delivery (CI/CD) — how it builds, tests, and ships. Default to stack-native /
   provider-native unless a constraint says otherwise.

Observability is roadmap (phase F) — record a one-line default (basic logs) and move on.
</decision-tree>

<cloud-rubric>
(phase D) Score the candidates the user surfaces on three weighted criteria — setup
difficulty is folded into operational cost as amortized upfront effort (automation
converts setup → lower ongoing ops; scoring them separately double-counts effort):

- Monetary cost — HIGH; tiebreaker when it conflicts with effort.
- Operational / effort cost — HIGH; ongoing ops with setup amortized in.
- Vendor lock-in — MED.

Tie rule: cheaper money wins over lower effort (time is more available than budget). These
weights are the user's stated priorities, not a lean toward any provider — gather the
candidate list during the interview and apply the weights to whatever the user is
considering. Standing preferences may override the weights per project; durable changes
promote back to standing preferences.
</cloud-rubric>

<outputs>
Write lazily — only once a decision is real.
- CLAUDE.md — update the brief with the decided architecture, stack, infra/cloud, and
  delivery as tight one-liners (ambient-light; it loads every turn). Link deeper
  rationale into docs/. Note any topology split flagged for `carve-repo`.
- CONTEXT.md — record survey findings and any new terms coined.
- docs/adr/NNNN-slug.md — emit an ADR when (and only when) a choice REVERSES a prior
  decision, or is a hard-to-reverse, surprising-in-hindsight trade-off (lock-in tech,
  architectural shape, deliberate deviation). Keep it to 1–3 sentences (format below).
Hand off to `scaffold-skills` once architecture, stack, infra/cloud, and delivery are decided.
</outputs>

<supporting-info>
## Altitude
Architecture only — coarse shape and load-bearing, hard-to-reverse choices. Detailed
module design is downstream (per-feature / `to-tasks`); writing IaC and provisioning is
execution (`scaffold-skills` / orchestrator); executing the repo split is `carve-repo`.
architect only MAKES the topology call.

## Standing preferences
If a navigate home with PREFERENCES.md exists, load it first (preferred languages, cloud
criteria weights, lock-in tolerance), skip settled branches, and promote durable new
choices at the end. (Its home — navigate-wide above projects, or a repo-local file — is
configurable; if none is set, skip this step.)

## ADRs — when and how
Offer one only when **all three** hold: **hard to reverse** (changing your mind later is
costly), **surprising without context** (a future reader sees the code and wonders "why on
earth this way?"), and **the result of a real trade-off** (genuine alternatives, picked for
specific reasons). The `change`/`audit` reversal case almost always qualifies. If it's easy
to reverse, unsurprising, or had no real alternative, skip it — it just lives in CLAUDE.md.

**Format.** ADRs live in `docs/adr/` with sequential numbering (`0001-slug.md`); create the
dir lazily on the first one, scan for the highest number and increment. The body is tiny:

```md
# {Short title of the decision}

{1–3 sentences: the context, what you decided, and why.}
```

A single paragraph is fine — the value is recording *that* a decision was made and *why*,
not filling sections. Add optional bits only when they genuinely help: a `Status` frontmatter
(`proposed | accepted | deprecated | superseded by ADR-NNNN`) when decisions get revisited,
**Considered Options** when the rejected alternatives are worth remembering, **Consequences**
when non-obvious downstream effects need calling out.

## Blast radius → topology
A real infra-vs-app blast-radius boundary (separate creds, lifecycle, permissions) is the
ONLY justification for a repo split, and even then it's deferred to `carve-repo`, fired
demand-driven. Default to one repo; record the decision, don't execute the split.
</supporting-info>
