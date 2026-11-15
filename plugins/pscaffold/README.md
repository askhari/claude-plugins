# pscaffold plugin

Product pipeline for apps that ship as iOS / iPadOS / Android / web clients backed by an API,
built mostly by Claude Code agents:

```
kickoff ──► architect ──► new-project ──► ┌─ spec ──► plan ──► orchestrate ─┐
(product)   (decisions)   (copier repo)   └──────── once per feature ───────┘
```

| Skill | Output |
|---|---|
| `kickoff` | `docs/brief.md`: problem, users, MVP, roadmap |
| `architect` | `docs/adr/*`, brief updates, `docs/copier-answers.yml` |
| `new-project` | A product repo generated from the template whose `just verify` passes |
| `spec` | `docs/specs/NNNN-slug/spec.md` for one roadmap item |
| `plan` | `docs/specs/NNNN-slug/tasks.md`: vertical slices with risk tier and worker model |
| `orchestrate` | One PR per task, implemented by the template's `implementer` agent |

`PREFERENCES.md` holds the standing decisions the interviews start from.

## Template

The skills operate products generated from the
[pscaffold copier template](https://github.com/askhari/pscaffold). They rely on the
template's **contract** (recipe names, document layout, task table, PR conventions, the
`implementer` agent), written down in the template's `CONTRACT.md`.

| Plugin version | Template contract |
|---|---|
| 0.x | 1 |

A generated product declares its contract in `.claude/pscaffold.json`; `new-project` and
`orchestrate` stop if it does not match.

## Install

```bash
claude plugin marketplace add askhari/claude-plugins
claude plugin install pscaffold@askharilabs
```

Products generated from the template already enable it in `.claude/settings.json`.
