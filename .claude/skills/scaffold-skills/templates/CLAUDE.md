# Project context

<!-- This file is loaded on every turn of every session in this project. -->
<!-- Keep it under ~500 tokens. Everything else belongs in skills or rules. -->

## Stack

- **Primary concern:** {{PRIMARY_CONCERN}}
- **Languages:** {{LANGUAGES}}
<!-- IF:HAS_FRAMEWORK -->
- **Framework:** {{FRAMEWORK}}
<!-- ENDIF -->
<!-- IF:USES_IAC -->
- **Infrastructure:** {{IAC_TOOL}} targeting {{CLOUD}}
<!-- ENDIF -->
<!-- IF:USES_CI -->
- **CI/CD:** {{CI_PLATFORM}}
<!-- ENDIF -->
- **Environment level:** {{PROD_LEVEL}}

## How to work in this repo

- Propose changes before making them. Small, isolated diffs over large refactors.
- Review your own changes before declaring done.
<!-- IF:USES_IAC -->
- For infrastructure changes, never run apply/destroy commands directly — require explicit confirmation.
<!-- ENDIF -->

## Hard rules

- Never commit secrets, `.env` contents, or credentials.
- Never modify generated files by hand (they have a header saying so).
<!-- IF:USES_BASH -->
- Shell scripts must pass `shellcheck` before commit. Disabled checks need a one-line comment explaining why.
<!-- ENDIF -->
<!-- IF:HAS_PROD -->
- This repo deploys to production. Treat all `main`/`master` branch changes as production-bound. Require explicit confirmation before any action that could reach prod.
<!-- ENDIF -->
<!-- IF:HAS_CUSTOM_RULES -->

### Project-specific rules
{{CUSTOM_RULES}}
<!-- ENDIF -->

## Where things live

- `.claude/skills/` — reusable workflows, invoked via `/<name>` or auto-triggered
- `.claude/rules/` — path-scoped conventions that auto-load when matching files are touched
- `.claude/settings.json` — permissions and hooks
- `.claude/README.md` — what's here and why, regenerate notes
