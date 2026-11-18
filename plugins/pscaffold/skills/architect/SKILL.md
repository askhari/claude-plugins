---
name: architect
description: Architecture interview for a product with a decided brief — settle the decisions the code cannot show (cross-cutting concerns, sync rules, infra target, risk profile) as ADRs and produce the copier answers for `new-project`. Second step of the pscaffold pipeline. Use after `kickoff`, on "architect", "decide the architecture", or to audit an existing product's architecture against its brief.
argument-hint: "[product directory] [new|change|audit]"
---

# architect

The stack is already decided by `PREFERENCES.md` (Expo universal app, Go or Rust API,
PostgreSQL, OpenAPI contract, deployment onto the shared platform, GitHub Actions, EAS). Do not re-open it. Your job is
the decisions that are product-specific and expensive to reverse, written down where agents
will find them.

## Modes

- `new` (default when there is no `.copier-answers.yml`): run every phase.
- `change`: a change is in hand. Read the repo first (CLAUDE.md, docs/adr, contract,
  infra), then run only the phases the change touches.
- `audit`: read the repo and walk every phase asking "is this still right for the brief at
  today's scale?". Recommend deltas.

## Before asking anything

Read `docs/brief.md`, `PREFERENCES.md` (`../../PREFERENCES.md` relative to this skill's directory), and
`docs/adr/`. If the brief is missing or undecided, send the user back to `kickoff`.

## Interview

One question at a time, each with your concrete recommendation. Skip what the preferences or
existing ADRs settle unless the brief gives a reason to deviate.

**A. Deviations from the preferences.** Does anything in the brief break a default (e.g. no
mobile, heavy realtime, regulated data, SEO-critical public pages that need a separate site)?
If not, say so and move on.

**B. Cross-cutting concerns** — the part agents get wrong when it is left implicit:
1. *Identity*: which sign-in methods for the MVP; single user vs organizations/tenants;
   roles. Remind the user that offering Google/Microsoft on iOS requires Sign in with Apple.
2. *Offline and sync*: which entities sync to the device; for each, the conflict rule
   (last-write-wins is the default; name the entities where it is wrong); what must never be
   editable offline (payments, unique handles, …).
3. *Data ownership and privacy*: personal data stored, retention, deletion on account removal,
   export obligations.
4. *Files and media*: whether users upload files (→ object storage), sizes.
5. *Notifications and background work*: push, email, scheduled jobs (→ a worker process?).
6. *Anything else the brief implies*: payments, i18n, search, realtime.

**C. Platform fit.** The target is settled: the shared platform from the preferences (API
container only; PostgreSQL, TLS and buckets come from the platform). Do not ask where it
runs. Settle only:
- the product's name on the platform (`platform_product`): recommend the slug without dashes,
  and ask whether the tenant already exists there under another name;
- what the product needs from the platform beyond a database: a bucket (when B.4 says files),
  a worker or scheduled job (B.5), more than the default 256 MB for the API container;
- anything the platform cannot give (another region, a managed database, heavy compute). Only
  then is it a deviation: write an ADR and tell the user the template will not generate it.
Only `dev` exists at first; record when `prod` should be added (first external user, first
TestFlight build).

**D. Risk profile and verification.** Decide, per area, how much human review a change needs
and what must always be tested:
- *High risk* (two reviews or explicit human approval): typically `apps/api/**/auth/**`,
  migrations, `infra/**`, anything touching payments or personal data.
- *Normal* (one review): product code.
- *Low* (green checks are enough): docs, copy, tests-only changes.
- *Critical test suites*: which suites `just test-affected` must always run when a path
  changes (e.g. auth tests whenever `auth/` changes).

## Outputs

Write only real decisions.

- `docs/adr/NNNN-slug.md` — one per decision that is hard to reverse, surprising without
  context, and the result of a real trade-off. Sequential numbers, 1–3 sentences each; add
  "Considered options" only when the rejected options are worth remembering. Typical
  outputs: sync conflict rules, tenancy model, what the product needs from the platform.
- `docs/brief.md` — append an `## Architecture` section with one-liners: deviations,
  cross-cutting decisions, platform needs, risk tiers, critical suites. Link the ADRs.
- `docs/copier-answers.yml` — the answers `new-project` will feed to copier:

  ```yaml
  project_name: "<Product name>"
  project_slug: "<kebab-case>"
  backend: go            # go | rust
  platform_product: "<slug without dashes>"   # the product's name on the shared platform
  github_owner: "<org or user>"
  bundle_id: "com.<owner>.<slug>"
  agent_name: "<git author name for agent commits>"
  agent_email: "<git author email for agent commits>"
  ```

  Take `agent_name` / `agent_email` from the developer's git config
  (`git config --get pscaffold.agentName`, `git config --get pscaffold.agentEmail`) when set;
  otherwise ask once, suggesting `<github_owner>-agent`. Never write the developer's own
  identity here: these keep agent commits apart from human ones.

In `change`/`audit` mode on an existing repo, also list the concrete follow-ups (template
update, CODEOWNERS change, new critical suite) and offer to hand them to `spec`/`plan`.

Finish by naming the next step: `new-project` (new) or `spec` (change).
