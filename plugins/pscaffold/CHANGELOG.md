# Changelog

All notable changes to the `pscaffold` plugin. Format:
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions follow `version` in
`.claude-plugin/plugin.json` and are tagged `pscaffold--vX.Y.Z` in this repo.

## [Unreleased]

## [0.3.0] - 2026-11-18

Speaks template contract **2** (pscaffold template `v0.2.x`).

### Changed
- **Breaking:** products deploy onto the shared platform. `PREFERENCES.md` drops Terraform
  and the DigitalOcean / own-VPS choice; `architect` no longer asks for an infrastructure
  target and writes `platform_product` instead of `infra_target`; `new-project` expects
  contract 2.
- `new-project` checklist: only `DEPLOY_HOST`, `DEPLOY_SSH_KEY` and `EXPO_TOKEN` as secrets,
  no Terraform state bucket, and the dev host steps are replaced by adding the tenant on the
  platform.
- `PREFERENCES.md` pins PostgreSQL to major 18, the platform's.

## [0.2.1] - 2026-11-16

Speaks template contract **1** (pscaffold template `v0.1.x`; agent identity needs `v0.1.1`).

### Added
- `architect` writes `agent_name` / `agent_email`, defaulting to the developer's
  `git config pscaffold.agentName` / `pscaffold.agentEmail`.
- `orchestrate` briefs workers with their model and task ID for `just agent-identity`, merges
  with merge commits only (`gh pr merge --auto --merge`) and reports models from the
  `Agent-Model` trailers.

## [0.2.0] - 2026-11-15

Speaks template contract **1** (pscaffold template `v0.1.x`).

### Changed
- Published from the askharilabs marketplace instead of the template repo.
- `new-project` renders the template's newest release tag (or `--vcs-ref` on request) with
  `uvx copier`; no local pscaffold checkout is needed.

### Added
- `new-project` and `orchestrate` check the product's `.claude/pscaffold.json` and stop when
  its contract version is not 1.

## [0.1.0] - 2026-11-15

### Added
- Pipeline skills `kickoff`, `architect`, `new-project`, `spec`, `plan`, `orchestrate`, and
  the standing `PREFERENCES.md`.
- `plan` assigns a worker model per task; `orchestrate` dispatches the template's
  `implementer` agent on it, judges PRs from evidence and escalates haiku → sonnet → opus →
  HITL.
