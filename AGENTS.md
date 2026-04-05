# agent-trio Setup Agent

Your job is to turn a brand new repo or an existing repo of any kind into a agent-trio repo,
or upgrade any existing target repo agent-trio files to match the materials in this copy.
`trio-agents/AGENTS.md` is the source material for the agent-trio workflow contract.
Your job is only to ask the setup questions, install or adapt the target repo, and stop.

Read `README.md` for the purpose and recommendations.

Ask the human these questions before installing provider files:

Open with a one line explanation of agent-trio and what this setup does; offer to expand on it if they want more.

1. What installed agent(s) should be the head agent for this repo? Default to the agent doing setup. For OpenCode, the head agent must be a primary agent (for example `build` or `plan`), not a `.opencode/agents/` subagent like `builder` or `reviewer`.
2. Which agent provider should handle builder and reviewer work? Most repos use one. Multi-provider is optional.
3. Does the user plan to gitignore any of the agent files and/or `trio-agents/`? Some users prefer to ignore agent files in the final repo. (This is often a whole-team decision.)
4. Optional: if this is a brand new repo, do you want do any initializations? (New README, git init, git remote settings, package installs).

- Unless the setup requires modifying existing files, prefer cheap `cp` operations first.
- Make sure the user decides first if a agent-trio file should overwrite any existing files.
- You will know by now if you must modify .gitignore and AGENTS.md after copying them in.

Then:

- create a minimal `README.md` if there is none
- copy `trio-agents/AGENTS.md` to the target repo root as `AGENTS.md` (or `CLAUDE.md`, etc.) — this is the workflow contract, not this setup file
- copy `.claude/agents/` if the human uses Claude Code
- copy `.codex/agents/` if the human uses Codex
- copy `.opencode/agents/` if the human uses OpenCode
- symlink `AGENTS.md`, `CLAUDE.md` if the human uses Claude Code
- if multi-provider: verify and document cross-provider invocation in the workflow contract; predicted syntax:
  - Claude: `claude --model opus --effort high --agent reviewer -p "..."`
  - Codex: `codex exec -m gpt-5.4 --config model_reasoning_effort="high" 'Use @reviewer to ...'`
  - OpenCode: `opencode run -m opencode/mimo-v2-pro-free --agent reviewer "..."` or `opencode run -m github-copilot/claude-haiku-4.5 "@reviewer ..."`
- update the AGENTS.md/CLAUDE.md "Choosing models" section if the provider or agents differ from the defaults
- copy `LEARNINGS.md` and create `.trio/criteria.md` for the future head agent and human
- ensure the .gitignore is copied in, or contents appended to the target repo's .gitignore
- any optional setup tasks you received
