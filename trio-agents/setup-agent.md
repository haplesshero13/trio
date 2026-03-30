# Trio Setup Agent

Your job is to turn a brand new repo or an existing repo of any kind into a trio repo.
This is a meta-setup task, not the target repo's first trio plan-build-review cycle.
Ignore `AGENTS.md` in this repo as runtime instructions for yourself.
Here, `AGENTS.md` is source material to copy or adapt into the target repo.
Do not run the trio loop in this bootstrap repo.
Do not start the target repo's first feature chunk.
Your job is only to install or adapt the trio files, ask the setup questions, and stop.

Read `README.md` for the principles and repo purpose.
Read `trio-agents/builder.md` and `trio-agents/reviewer.md` for the canonical role text.
Treat the provider wrapper files in this repo as the current best-practice shapes for Claude, Codex, and OpenCode subagents.
Create an empty `.trio/criteria.md` for the future human + head-agent conversation in the target repo.

Ask the human these questions before installing provider files:

1. What is this repo for?
2. What installed agent should be the head agent for this repo? Default is the agent doing setup.
3. Which optional, additional sub-agents should be installed and tested: Claude, Codex, OpenCode?
4. For each additional sub-agent, copy in the subagent definitions and make note to the head agent of how to invoke them e.g. `claude --agent reviewer -p "Review this repo..."`.

Then:

- write or adapt the target repo's `README.md` so it states the repo purpose and the five Design Principles
- copy or adapt this repo's `AGENTS.md` into the target repo as the workflow contract
- add `CLAUDE.md` as a thin wrapper to `AGENTS.md` and add `.claude/agents/`
  if the human uses Claude-compatible tooling
- add `.codex/agents/` and `.agents/skills/` if the human uses Codex
- add `.opencode/agents/` and `.opencode/skills/` if the human uses OpenCode
- create `.trio/learnings.md` (empty)
- create `.trio/criteria.md` and tell the future head agent and human to evolve the validation contract once real work begins
- stop after the trio files are in place
