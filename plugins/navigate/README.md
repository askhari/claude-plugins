# navigate

**Fuzzy idea → running, agent-built software.**

`navigate` is a [Claude Code](https://claude.com/claude-code) plugin: a pipeline of
scoped *interview + templating* skills that take a vague idea all the way to the durable
artifacts autonomous agents build from — a product brief, ADRs, PRDs, and a git-native
task queue.

It works inside a **single repo**. Zoom (what → architecture → tech → feature → tasks) is
expressed by the skill pipeline; blast-radius (infra vs app) stays a *decision* the
`architect` skill makes, not an upfront assumption. See [`DESIGN.md`](DESIGN.md) for the
full design record.

## The pipeline

| # | Skill | What it does | Writes |
|---|-------|--------------|--------|
| 1 | **navigate-brainstorm** | Product tier-0 interview: problem, who/why, MVP, roadmap | `CLAUDE.md`, `CONTEXT.md` |
| 2 | **architect** | Architecture, stack/languages, infra & cloud, CI/CD; decides if/when topology splits | updates `CLAUDE.md` |
| 3 | **scaffold-skills** | Generates the project-specific `.claude/` set (skills, rules, safety hooks) from the decided stack | `.claude/` |
| 4 | **to-prd** | Feature zoom: takes ONE roadmap item to a reviewable PRD (modules, user stories, testing) | `docs/prd/NNNN-slug.md` |
| 5 | **to-tasks** | Layer slice: cuts ONE PRD into AFK/HITL vertical-slice tracer-bullet tasks | `tasks/` |

Steps **1–3 run once per project**. **`to-prd → to-tasks` is the repeating inner loop** —
pick a roadmap item, PRD it, slice it — realizing the brief one feature at a time. An
orchestrator of your own (e.g. a Rust/Go binary) drains `tasks/`.

Each skill runs a relentless **one-question-at-a-time** interview that recommends an answer
at every step, loads any standing preferences first, and skips every branch they already
settle. Later skills *synthesize* from the rich brief instead of re-interviewing.

## Install

navigate is published in the **askharilabs** marketplace
([askhari/claude-plugins](https://github.com/askhari/claude-plugins)). From inside Claude Code:

```
/plugin marketplace add askhari/claude-plugins
/plugin install navigate@askharilabs
```

The first command registers the marketplace; the second installs the
`navigate` plugin and its five skills. Run `/plugin` any time to browse, enable, or remove
installed plugins.

To update later:

```
/plugin marketplace update askharilabs
```

## Usage

Once installed, the skills trigger from natural language or by name:

- Start a new project: **"navigate"** or *"brainstorm what to build"* → `navigate-brainstorm`
- Decide the build: **"architect"** or *"decide the stack/architecture"* → `architect`
- Generate config: **"scaffold the skills"** / *"generate .claude"* → `scaffold-skills`
- Spec a feature: **"PRD this"** / *"zoom into &lt;roadmap item&gt;"* → `to-prd`
- Slice into tasks: **"to-tasks"** / *"slice this PRD"* → `to-tasks`

Run them in order on a fresh repo, then loop `to-prd → to-tasks` per feature.

## Layout

```
.claude-plugin/
  plugin.json             # the navigate plugin manifest (v0.1.0)
.claude/skills/
  navigate-brainstorm/    # 1 · product brief
  architect/              # 2 · architecture + stack
  scaffold-skills/        # 3 · generates .claude/ (bundles a template library)
  to-prd/                 # 4 · feature PRD
  to-tasks/               # 5 · task slices
docs/                     # per-skill design briefs
DESIGN.md                 # design record
```

## License

[MIT](LICENSE) © askharilabs
