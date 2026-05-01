# agent trio

A minimal, all natural language bootstrap for brand new or existing repos of any kind that want to follow principled agentic coding. Zero code.

Once upon a time, humans hand-wired computers to program them. Thankfully, someone invented ~~silicon chips~~ coding agents.

**agent trio** is not a swarm framework. It does not optimize for agent count or lines of code output. It is for introducing _just the right amount of friction_. You should spend tokens to maximize judgement and quality, not output or wasteful rework. **agent trio** helps you do that. See the [intro blog post](https://averyyen.dev/2026/04/01/introducing-agent-trio.html) for more.

**How is this different from `/review`?** `/review` is a one-shot critique of git state; agent-trio holds out `.trio/criteria.md` _before_ building, so the review phase checks the implementation against criteria set before it existed.

**How is this different from [openai/codex-plugin-cc](https://github.com/openai/codex-plugin-cc)?** That plugin ships cross-provider adversarial review as a skill; agent-trio is the plan → build → review → learn loop around it, and the reviewer can invoke `/codex:adversarial-review` as one of its inputs.

**Recommendations.** I recommend you select a more efficient (but not _deliberately cheap_) builder model, and a similar-or-stronger reviewer model. Keep your expensive tokens for reasoning and judgement. You could for example use Codex + GPT-5.4 (xhigh) as the planner and reviewer, and GPT-5.4 (med) for the builder. Claude Code can spawn Opus reviewer and Sonnet builder subagents. Or go rainbow providers — you just need to configure your agents accordingly.

## Installation

Agent-trio installs as a plugin that injects the workflow contract and agents into your coding agent session.

**Claude Code:**

Register the plugin marketplace, then install:

```bash
/plugin marketplace add haplesshero13/agent-trio
/plugin install agent-trio@trio-marketplace
```

Or add to your project's `.claude/settings.json`:

```json
{
  "plugins": ["agent-trio@git+https://github.com/haplesshero13/agent-trio.git"]
}
```

**Codex:**

```bash
codex plugin marketplace add haplesshero13/agent-trio
```

Then open Codex, run `/plugins`, select **Agent Trio**, and install it. For local
development from this checkout, run `codex plugin marketplace add .`.

The skill operates in `consult`, `plan`, or `execute` mode depending on what you ask — it will not create planning files for read-only questions or simple guidance. Builder and reviewer agents are available when Codex runs inside a checkout; without them, the head instance acts locally and still honors the full artifact contract.

See `.codex/INSTALL.md` for full details and examples.

**OpenCode:**

If `agent-trio` is available from npm:

Add agent-trio to the `plugin` array in your `opencode.json`:

```json
{
  "plugin": ["agent-trio"]
}
```

For local development or git-checkout installation, clone the repo and add a
global shim under `~/.config/opencode/plugins/`; see `.opencode/INSTALL.md`.

Restart OpenCode. The plugin registers the repo-root `skills/` directory and
the `.opencode/agents/` wrappers, so `using-agent-trio`, `@builder`, and
`@reviewer` become available.

See `.opencode/INSTALL.md` for details.

## Interactive setup

Once agent-trio is installed, run the setup flow to pick builder/reviewer models and peer-review preferences without hand-editing config:

- **Claude Code:** `/agent-trio:setup`
- **OpenCode / Codex / any other platform:** ask the head agent to "run agent-trio setup" — it will read `setup/agent-trio-setup.md` and walk you through the same choices.

The setup flow only writes to files it tells you about first; see `setup/agent-trio-setup.md` for the full list of files it may create or modify per platform.

## Quick file guide

- `AGENTS.md`: developer guide — repo structure, how to make changes, validation checklist
- `skills/using-agent-trio/SKILL.md`: the workflow contract (imperative, with first-session bootstrap and delegation)
- `agents/builder.md`, `agents/reviewer.md`: the builder/reviewer agent definitions
- `.trio/learnings.md`: the checked-in lessons that still change how the next loop runs
- `.trio/plan.md`, `.trio/handoff.md`, `.trio/review.md`: the gitignored agent coordination files
- `.claude/agents/`, `.codex/agents/`: provider wrappers around the roles
- `.codex-plugin/`, `.agents/plugins/`: Codex plugin manifest and marketplace metadata
- `.opencode/`: OpenCode plugin — `plugins/agent-trio.js`, `package.json`, `INSTALL.md`
- `.claude-plugin/`: plugin manifest for marketplace distribution
- `commands/setup.md`: `/agent-trio:setup` slash command for Claude Code
- `setup/agent-trio-setup.md`: platform-neutral interactive setup instructions
