# agent-trio setup

This is the platform-neutral script for the `agent-trio setup`
flow. It is invoked by:

- **Claude Code:** `/agent-trio:setup` — the slash command in
  `commands/setup.md` redirects here.
- **OpenCode / Codex / other platforms:** the user asks the head agent to
  "run agent-trio setup" and the agent reads this file and follows it.

You (the agent executing this flow) walk the user through a short series of
questions, then write exactly the files listed in **Generated output** below.
Do not touch files outside that list. If a generated file already exists,
show the diff and ask the user to confirm before overwriting.

## Run mode

If you have a native structured-question tool (Claude Code's
`AskUserQuestion`, OpenCode's `ask`, Codex's interactive prompts), use it so
the user can pick from choices. Otherwise, ask in plain text and accept free
form answers.

## Steps

### 1. Target platform

Ask: "Which platform are you configuring agent-trio for?"

Options:

- `claude-code` — Claude Code.
- `opencode` — OpenCode.ai.
- `codex` — OpenAI Codex CLI.

Default to your current agent. Branches below are keyed off this answer.

### 2. Builder model

Ask: "Which model should the builder use?"

Suggest the platform's current efficient-but-not-cheap model as the default.
Accept any model identifier the user provides — do not validate against an
allowlist, since model IDs change frequently.

### 3. Reviewer model

Ask: "Which model should the reviewer use?"

Recommend a similar-or-stronger model than the builder. If the user picks
the same model as the builder, note that they will lose adversarial-review
benefits and ask them to confirm.

### 4. Peer / cross-provider review

Ask: "Do you want the reviewer to be a peer or cross-provider agent?"

Options:

- `same` — reviewer runs on the same model as the builder
  (simpler, weaker adversarial signal).
- `peer` — reviewer runs on a different model from the builder,
  same provider.
- `cross-provider` — reviewer runs as a different agent and model (e.g.,
  invoking Codex from Claude Code via `/codex:adversarial-review`).

Record the choice. If `cross-provider`, help the user test and debug it.

### 5. Builder / reviewer agent access in the target repo

Ask: "Should the target repo have its own `.claude/agents/`, `.codex/agents/`,
or `.opencode/` wrappers vendored in, or rely on plugin-level discovery?"

Options:

- `plugin-only` — do not vendor wrappers; rely on the plugin to expose
  `@builder` / `@reviewer`. Best for repos that already have the plugin
  installed.
- `vendored` — copy the thin wrappers (and the `agents/*.md` templates they
  reference) into the target repo. Best when the target repo may be opened
  outside a checkout of agent-trio.

### 6. Write files

Using the answers, write only the files listed for the chosen platform in
**Generated output** below. Before writing each file:

1. If the file already exists and you would change it, show the proposed
   diff.
2. Ask the user to confirm.
3. Only then write.

Never edit files that are not on the per-platform list without first
showing the user what would change and why.

### 7. Summarize

Print a short summary: platform, builder model, reviewer model, peer
choice, access mode, and the list of files you wrote. Point the user at
the agent-trio workflow (`.trio/plan.md` → `.trio/handoff.md` → `.trio/review.md`) as next
steps.

## Generated output

The setup flow may create or modify only the files listed here for the
selected platform. Anything else must be proposed and confirmed first.

### Claude Code (`claude-code`)

Setup may write:

- `.claude/agents/builder.md` — thin wrapper; sets `model:` frontmatter to
  the builder choice and points at `agents/builder.md`.
- `.claude/agents/reviewer.md` — thin wrapper; sets `model:` frontmatter to
  the reviewer choice and points at `agents/reviewer.md`.
- `.claude/settings.local.json` — may add a note under a `comments` key
  documenting the cross-provider reviewer command, if chosen. Never
  overwrites existing keys; merges additively.

If the user chose `plugin-only`, setup skips the two `.claude/agents/*.md`
files and only records the model choice as a comment in
`.claude/settings.local.json`.

### OpenCode (`opencode`)

Setup may write:

- `opencode.json` at the target repo root — adds agent-trio to the `plugin`
  array if npm installation is being used, and records the builder/reviewer
  model choices under `agent.builder.model` and `agent.reviewer.model`. Never
  rewrites existing unrelated keys; merges additively.
- `.opencode/agents/builder.md` — _only if_ the user chose `vendored`. Thin
  wrapper that reads `agents/builder.md` (vendored alongside). In
  `plugin-only` mode, no `.opencode/agents/` files are written.
- `.opencode/agents/reviewer.md` — same as above.

Vendoring requires copying `agents/builder.md` and `agents/reviewer.md`
into the target repo too, since the wrappers reference them by path. Setup
must list these files explicitly and confirm before copying.

If the user is using a git checkout instead of npm, setup should print the
global shim instructions from `.opencode/INSTALL.md` and ask before modifying
`~/.config/opencode/plugins/agent-trio.js`; that file is outside the target
repo and is not part of the default generated output.

### Codex (`codex`)

Setup may write:

- `.agents/plugins/marketplace.json` — adds a local marketplace entry for
  `agent-trio` when configuring this repo as a Codex plugin marketplace.
- `.codex-plugin/plugin.json` — records plugin metadata and points Codex at
  `./skills/`. Setup should not duplicate skill content here.
- `.codex/agents/builder.toml` — thin TOML wrapper. Records the builder
  model in the `description` field since TOML wrappers do not currently
  carry a typed model selector; the actual model is picked at invocation
  time. Only written in `vendored` mode.
- `.codex/agents/reviewer.toml` — same as above.
- `agents/builder.md` — _only if_ the user chose `vendored`. The TOML
  wrappers reference this file by path, so it must exist in the target repo
  for the agents to load. Setup must list this file explicitly and confirm
  before copying.
- `agents/reviewer.md` — same as above.
- `.codex/INSTALL.md` — never modified by setup; leave as-is.

Vendoring requires copying `agents/builder.md` and `agents/reviewer.md`
into the target repo too, since the `.toml` wrappers reference those files
by path. Setup must list these files explicitly and confirm before copying.

In `plugin-only` mode on Codex, setup prints the marketplace install flow:
`codex plugin marketplace add <source>`, then open `/plugins`, select
**Agent Trio**, and install it. For older Codex builds without marketplace
support, setup may print the direct `~/.codex/skills/using-agent-trio` symlink
fallback from `.codex/INSTALL.md`.

## Files setup will never touch

- `skills/using-agent-trio/SKILL.md` — the workflow contract; human-edited
  only.
- `agents/builder.md` / `agents/reviewer.md` — authoritative role prompts;
  human-edited only.
- `.trio/learnings.md` — written by the head agent during the learn phase.
- `.trio/plan.md`, `.trio/handoff.md`, `.trio/review.md`, `.trio/criteria.md` — loop
  artifacts owned by plan/build/review phases, not by setup.

If the user asks setup to modify any of these, refuse and explain that
setup is scoped to configuration, not workflow artifacts.
