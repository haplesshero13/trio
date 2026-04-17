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

**OpenCode:**

Add to your `opencode.json`:

```json
{ "plugin": ["agent-trio@git+https://github.com/haplesshero13/agent-trio.git"] }
```

See `.opencode/INSTALL.md` for details.

**Codex:**

```bash
git clone https://github.com/haplesshero13/agent-trio.git ~/.codex/agent-trio
mkdir -p ~/.agents/skills && ln -s ~/.codex/agent-trio/skills ~/.agents/skills/agent-trio
```

See `.codex/INSTALL.md` for details.

**Cursor:**

Search for "agent-trio" in the plugin marketplace, or see `.cursor-plugin/plugin.json` for manual setup.

## Quick file guide

- `AGENTS.md`: developer guide — repo structure, how to make changes, validation checklist
- `skills/using-agent-trio/SKILL.md`: the workflow contract (imperative, with first-session bootstrap and delegation)
- `agents/builder.md`, `agents/reviewer.md`: the builder/reviewer agent definitions
- `LEARNINGS.md`: the checked-in lessons that still change how the next loop runs
- `PLAN.md`, `HANDOFF.md`, `REVIEW.md`: the gitignored agent coordination files
- `.opencode/agents/`, `.claude/agents/`, `.codex/agents/`: provider wrappers around the roles
- `.claude-plugin/`, `.cursor-plugin/`: plugin manifests for marketplace distribution