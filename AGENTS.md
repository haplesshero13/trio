# agent-trio — Developer Guide

This is a skill/plugin repo. The workflow contract is `skills/using-agent-trio/SKILL.md`. Everything else is either a wrapper, a redirect, or install infrastructure.

## Making changes

1. Edit content in `skills/` or `agents/`.
2. Do **not** duplicate content into wrappers — wrappers say "Read `agents/builder.md` and follow it exactly."
3. Run `claude plugin validate .` to check that `.claude-plugin/plugin.json` is valid.
4. Run `HOME=/tmp/agent-trio-codex-home codex plugin marketplace add .` to check that `.agents/plugins/marketplace.json` is valid.
5. If you add a new agent or skill, update the relevant plugin manifest.

## Validation checklist

- `claude plugin validate .` passes
- `HOME=/tmp/agent-trio-codex-home codex plugin marketplace add .` passes
- `skills/using-agent-trio/SKILL.md` is the only full copy of the workflow contract
- `agents/builder.md` and `agents/reviewer.md` are self-contained
- All wrappers point to content, not to other wrappers
- `package.json` version matches `.claude-plugin/plugin.json` version

## What to put where

- Workflow rules → `skills/using-agent-trio/SKILL.md`
- Codex plugin metadata → `.codex-plugin/plugin.json` and `.agents/plugins/marketplace.json`
- Current durable lessons → .trio/learnings.md
- Start instructions → README
- Code style → linter
- Anything that could go stale → search the repo
- Interactive setup content → `setup/agent-trio-setup.md` (platform-neutral; wrappers like `commands/setup.md` must only point at it)

## Setup flow and file ownership

`setup/agent-trio-setup.md` is the single source of truth for what the
`agent-trio setup` flow may write. If you add a platform or a new
configurable choice, update that file — not the slash command wrapper and
not individual platform docs. The per-platform **Generated output** table
there is intentionally restrictive: setup must never modify files outside
the listed set without prompting the user first.
