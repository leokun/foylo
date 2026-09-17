# Foylo

Family coordination and check-in app. Current phase: design and decisions, no implementation yet.
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
