# Learnings

Keep only lessons that change how the next update is planned, built,
reviewed, or validated. Rewrite, merge, and prune continuously.

Source control will track versions of this document, so keep it fresh.

## Current learnings

### One copy per concept

The workflow contract lives in `skills/using-agent-trio/SKILL.md` — nowhere else. Agent definitions live in `agents/` — nowhere else. Wrappers are one-line pointers, not copies. Duplication creates drift and confusion about which version is authoritative.
**Why:** Multiple copies of builder/reviewer content led to a 5-way duplication problem that the reviewer flagged. Collapsing to one copy eliminated an entire class of bugs.
**How to apply:** Never duplicate content into wrappers. If a wrapper needs more than "Read X and follow it exactly," the content belongs in the file.

### Skill must be imperative, not descriptive

The original `trio-agents/AGENTS.md` described the loop in third person. The reviewer flagged that this meant the plugin could load and still do nothing — the agent reads a description but doesn't know what to do. Rewriting as imperative ("You are the head agent. Write PLAN.md. Delegate to builder.") made the skill effective.
**Why:** A plugin that loads but doesn't change agent behavior is dead weight.
**How to apply:** When writing agent instructions, address the agent directly with commands, not descriptions of how things work.

### Plugin manifests are per-platform, not generic

`.claude-plugin/plugin.json` does not support an `agents` field — it caused validation to fail. Each platform discovers agents differently (Claude Code from `.claude/agents/`, OpenCode from plugin config hook, Codex from `~/.agents/skills/`). Don't assume cross-platform consistency in manifest schemas.
**Why:** A single validation failure blocks the entire plugin from loading.
**How to apply:** Run `claude plugin validate .` after any manifest change. Don't add fields that aren't in the platform's schema.

### Hooks with no runtime purpose are dead code

The original `hooks/` directory had a `run-hook.cmd` that was a documented no-op. The real injection happened via the plugin system. Three files and two manifest entries for something that does nothing violates the minimalism goal.
**Why:** Dead code confuses contributors and bloats the repo.
**How to apply:** Don't ship forward-compatible placeholders. Add the hook when it actually does something.

### AGENTS.md is for developers, README is for consumers

AGENTS.md describes how to modify this repo. README describes why you'd use it and how to install it. Mixing setup-agent instructions into AGENTS.md made it unclear who the audience was.
**Why:** The repo is now a skill/plugin repo, not just a template. The audience for "how to work on this" is different from "how to use this."
**How to apply:** AGENTS.md = developer guide. README = consumer-facing install and rationale.

### Codex works as a peer reviewer via `codex exec`

A one-shot `codex exec --full-auto --skip-git-repo-check` with a reviewer prompt (read the loop artifacts, verify claims by running commands, write REVIEW.md) produces a real adversarial review. It caught a criterion/repo mismatch the same-model reviewer would likely have missed.
**Why:** Adversarial review across model families surfaces a different class of findings than same-family review.
**How to apply:** For loops where the stakes or scope justify it, dispatch Codex (or any peer runtime) as the reviewer instead of `@reviewer`. The head keeps owning PLAN, criteria, and LEARNINGS.

### Criteria and repo shape drift together

When you delete a concept from the repo, audit `.trio/criteria.md` in the same loop. A stale criterion (e.g. "grep for trio-agents returns nothing across all .md") can fail against a repo that is otherwise correct, because LEARNINGS.md intentionally keeps historical mentions.
**Why:** The reviewer runs the criterion exactly as written; over-broad criteria cause false REJECTs.
**How to apply:** When retiring a concept, scope the matching criterion (and its grep) to where the concept should no longer appear, not everywhere.
