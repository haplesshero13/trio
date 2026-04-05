# agent trio

A minimal, all natural language bootstrap for brand new or existing repos of any kind that want to follow principled agentic coding. Zero code.

Once upon a time, humans hand-wired computers to program them. Thankfully, someone invented ~~silicon chips~~ coding agents.

**agent trio** is not a swarm framework. It does not optimize for agent count or lines of code output. It is for introducing _just the right amount of friction_. You should spend tokens to maximize judgement and quality, not output or wasteful rework. **agent trio** helps you do that. See the [intro blog post](https://averyyen.dev/2026/04/01/introducing-agent-trio.html) for more.

## How to use

Use this repo by booting up your agent in this repo (one that reads AGENTS.md or CLAUDE.md) and tell it the target repo path you want to turn into a agent-trio repo. It will ask you whether this is a fresh install or upgrade, and what agents you use, etc.

**Recommendations.** I recommend you select a more efficient (but not _deliberately cheap_) builder model, and a similar-or-stronger reviewer model. Keep your expensive tokens for reasoning and judgement. You could for example use Codex + GPT-5.4 (xhigh) as the planner, and GPT-5.4 (med) for the builder and reviewer subagents. Claude Code can spawn Opus reviewer and Sonnet builder subagents. Or go rainbow providers; you just need to let your setup agent know.

## Quick file guide

- `AGENTS.md`: the setup directive for turning a repo into a agent-trio repo
- `trio-agents/AGENTS.md`: the agent-trio head agent + subagent contracts
- `trio-agents/builder.md`: the builder subagent
- `trio-agents/reviewer.md`: the reviewer subagent
- `LEARNINGS.md`: the checked-in lessons that still change how the next loop runs
- `PLAN.md`, `HANDOFF.md`, `REVIEW.md`: the gitignored agent coordination files
- `.opencode/agents/`, `.claude/agents/`, `.codex/agents/`: provider wrappers around the roles
