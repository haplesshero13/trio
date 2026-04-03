# agent-trio Setup Agent

Your job is to turn a brand new repo or an existing repo of any kind into a agent-trio repo,
or upgrade any existing target repo agent-trio files to match the materials in this copy.
`trio-agents/AGENTS.md` is the source material for the agent-trio workflow contract.
Your job is only to ask the setup questions, install or adapt the target repo, and stop.

Read `README.md` for the purpose and recommendations.

Ask the human these questions before installing provider files:

1. What installed agent(s) should be the head agent for this repo? Default to the agent doing setup. For OpenCode, the head agent must be a primary agent (for example `build` or `plan`), not a `.opencode/agents/` subagent like `builder` or `reviewer`.
2. Which optional, additional agent providers should be installed and tested: Claude, Codex, OpenCode?
3. Ask which agent should be the repo's head agent, and which provider agents to install; most repos only need one, extras are optional for cross-provider convergence.
4. Does the user plan to gitignore any of the agent files and/or `trio-agents/`? Some users prefer to ignore agent files in the final repo. (This is often a whole-team decision.)
5. Optional: if this is a brand new repo, do you want do any initializations? (New README, git init, git remote settings, package installs).

- Unless the setup requires modifying existing files, prefer cheap `cp` operations first.
- Make sure the user decides first if a agent-trio file should overwrite any existing files.
- You will know by now if you must modify .gitignore and AGENTS.md after copying them in.

Then:

- create a minimal `README.md` if there is none
- copy this repo's `trio-agents/AGENTS.md` into the target repo _root_ as the workflow contract, naming the target file for the head agent (`AGENTS.md`, `CLAUDE.md`, etc.). do _not_ put `trio-agents/AGENTS.md` in the target repo, use the root
- copy `.claude/agents/` if the human uses Claude Code
- copy `.codex/agents/` if the human uses Codex
- copy `.opencode/agents/` if the human uses OpenCode
- symlink `AGENTS.md`, `CLAUDE.md` if the human uses Claude Code
- copy `LEARNINGS.md` and create `.trio/criteria.md` for the future head agent and human
- ensure the .gitignore is copied in, or contents appended to the target repo's .gitignore
- any optional setup tasks you received
