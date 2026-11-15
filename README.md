# askharilabs Claude Code plugins

A [Claude Code](https://code.claude.com) plugin marketplace.

| Plugin | What it does |
|---|---|
| [`navigate`](plugins/navigate) | From a fuzzy idea to agent-built software: brainstorm → architect → scaffold-skills → to-prd → to-tasks, for any stack. |
| [`pscaffold`](plugins/pscaffold) | Product pipeline on the [pscaffold](https://github.com/askhari/pscaffold) template (Expo + OpenAPI + Go/Rust API): kickoff → architect → new-project → spec → plan → orchestrate. |

## How the repositories relate

```
askhari/claude-plugins (this repo)         askhari/pscaffold
  plugins/navigate     standalone
  plugins/pscaffold    ── relies on ──────►  CONTRACT.md
                                             template/ + copier.yml
                                                    │ copier copy / update
                                                    ▼
                       ── installed in ───►  your product repos
```

- **navigate** is self-contained and works on any stack; it does not use the pscaffold
  template.
- **pscaffold** (the plugin) holds the skills; **pscaffold** (the template repo) holds what
  the skills generate and operate. They meet at the template's versioned
  [`CONTRACT.md`](https://github.com/askhari/pscaffold/blob/main/CONTRACT.md): the plugin
  speaks one contract version, each product declares the one it was generated with, and the
  skills stop on a mismatch.
- **Products** pull from both: copier fetches template releases, Claude Code installs the
  plugin from this marketplace. Neither repo keeps any record of the products that use it.

## Install

```bash
claude plugin marketplace add askhari/claude-plugins
claude plugin install pscaffold@askharilabs     # or navigate@askharilabs
```

Enable only the plugin a project uses: both ship an `architect` skill, for different
pipelines.

## Developing

```bash
claude --plugin-dir plugins/pscaffold     # try local changes without installing
claude plugin validate .                  # marketplace manifest
claude plugin validate plugins/<name>     # one plugin
```

Bump `version` in a plugin's `.claude-plugin/plugin.json` when it changes, so installed
copies pick up the update, add the entry to its `CHANGELOG.md`, and tag the release with
`claude plugin tag plugins/<name>` (creates `<name>--vX.Y.Z`). Changes to `pscaffold` that alter what it expects from
the template follow the template's `CONTRACT.md`.

## License

[MIT](LICENSE)
