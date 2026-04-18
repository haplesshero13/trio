# Installing agent-trio for Codex

Enable agent-trio skills in Codex via native skill discovery.

## Prerequisites

- Git

## Installation

1. **Clone the agent-trio repository:**

   ```bash
   git clone https://github.com/haplesshero13/agent-trio.git ~/.codex/agent-trio
   ```

2. **Create the skills symlink:**

   ```bash
   ln -s ~/.codex/agent-trio/skills/using-agent-trio ~/.codex/skills/using-agent-trio
   ```

   **Windows (PowerShell):**

   ```powershell
   cmd /c mklink /J "$env:USERPROFILE\.codex\skills\using-agent-trio" "$env:USERPROFILE\.codex\agent-trio\skills\using-agent-trio"
   ```

3. **Restart Codex** (quit and relaunch) to discover the skills.

## Verify

```bash
ls -la ~/.codex/skills/using-agent-trio
```

You should see a symlink (or junction on Windows) pointing to the agent-trio skill directory.

## How it works in Codex

**Global skill (discoverable everywhere):**
The symlink makes `using-agent-trio` available in any working directory. Codex loads it automatically when appropriate. The skill operates in one of three modes — `consult`, `plan`, or `execute` — so it will not hijack read-only sessions or create planning files when you just ask a question.

**Repo-local agent wrappers (only in a checkout):**
`.codex/agents/builder.toml` and `.codex/agents/reviewer.toml` are thin wrappers that point to `agents/builder.md` and `agents/reviewer.md`. They are only available when Codex runs inside a clone of this repo. If you want to use the builder/reviewer agents in your own project, you must copy **both** the `.toml` wrappers (`from .codex/agents/`) **and** the markdown templates (`from agents/builder.md` and `agents/reviewer.md`) into the corresponding locations in your project. Copying only the `.toml` wrappers is not sufficient — they will fail to load because `agents/builder.md` and `agents/reviewer.md` will not exist.

**Without repo-local agents:**
If builder/reviewer wrappers are not present, Codex acts as builder or reviewer locally, still honoring the full artifact contract (`PLAN.md`, `.trio/criteria.md`, `HANDOFF.md`, `REVIEW.md`).

## Updating

```bash
cd ~/.codex/agent-trio && git pull
```

Skills update instantly through the symlink.

## Uninstalling

```bash
rm ~/.codex/skills/using-agent-trio
```

Optionally delete the clone: `rm -rf ~/.codex/agent-trio`