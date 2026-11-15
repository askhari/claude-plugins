# askharilabs Claude Code plugins

A [Claude Code](https://code.claude.com) plugin marketplace.

| Plugin | What it does |
|---|---|
| [`navigate`](plugins/navigate) | From a fuzzy idea to agent-built software: brainstorm → architect → scaffold-skills → to-prd → to-tasks, for any stack. |
| [`pscaffold`](plugins/pscaffold) | Product pipeline on the [pscaffold](https://github.com/askhari/pscaffold) template (Expo + OpenAPI + Go/Rust API): kickoff → architect → new-project → spec → plan → orchestrate. |

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

Bump `version` in a plugin's `.claude-plugin/plugin.json` when its behaviour changes, so
installed copies pick up the update. Changes to `pscaffold` that alter what it expects from
the template follow the template's `CONTRACT.md`.

## License

[MIT](LICENSE)
