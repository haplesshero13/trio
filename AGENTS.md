# agent-trio — Developer Guide

This is a skill/plugin repo. The workflow contract is `skills/using-agent-trio/SKILL.md`. Everything else is either a wrapper, a redirect, or install infrastructure.

## Making changes

1. Edit content in `skills/` or `agents/`.
2. Do **not** duplicate content into wrappers — wrappers say "Read `agents/builder.md` and follow it exactly."
3. Run `claude plugin validate .` to check that `.claude-plugin/plugin.json` is valid.
4. If you change the skill, the OpenCode plugin (`agent-trio.js`) reads it at runtime — no rebuild needed.
5. If you add a new agent or skill, update the relevant plugin manifest.

## Validation checklist

- `claude plugin validate .` passes
- `skills/using-agent-trio/SKILL.md` is the only full copy of the workflow contract
- `agents/builder.md` and `agents/reviewer.md` are self-contained
- All wrappers point to content, not to other wrappers
- `package.json` version matches `.claude-plugin/plugin.json` version

## What to put where

- Workflow rules → `skills/using-agent-trio/SKILL.md`
- Current durable lessons → LEARNINGS.md
- Start instructions → README
- Code style → linter
- Anything that could go stale → search the repo
