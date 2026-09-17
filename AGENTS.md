# Foylo

Family coordination and check-in app. Current phase: design and decisions, with isolated compatibility prototypes. No production implementation yet.
Read `docs/README.md` before exploring: it indexes the design dossier and lists the open decisions in order.

## Conventions

- Everything in this repo is written in English: docs, ADRs, code, commits, issues.
- Never use the em dash character. Use a colon, comma, or hyphen instead.

## Agent skills

### Issue tracker

Issues live in this repo's GitHub Issues, driven with the `gh` CLI. See `docs/agents/issue-tracker.md`.

### Triage labels

Default vocabulary: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context layout: `CONTEXT.md` at the repo root and ADRs in `docs/adr/`. See `docs/agents/domain.md`.

## Git branches and worktrees

- Use `main` as the integration branch. Create a branch or worktree only when the user requests isolation; reuse an existing branch for the same task.
- Name work branches `<type>/<short-kebab-case-description>`. Allowed types: `feat`, `fix`, `hotfix`, `docs`, `chore`, `refactor`, `perf`, `test`, `ci`, `build`, `revert`. When an issue exists, start the description with its number, for example `fix/42-session-expiry`.
- The prefix describes the change. Tool-generated prefixes such as `t3Code/` or `t3code/`, generic worktree prefixes and random identifiers are forbidden. Rename an automatically created nonconforming branch before its first commit or push.
- When consolidation is requested, inventory unique commits and uncommitted files before merging. Preserve unique work, verify the result on `main`, and remove only branches whose work has been integrated or explicitly preserved.
- Keep local worktree directories out of tracked files. Never stage an embedded worktree as a gitlink. Preserve local environment files outside commits.
