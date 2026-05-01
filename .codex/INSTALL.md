# Installing agent-trio for Codex

Enable agent-trio in Codex via the plugin marketplace. The marketplace exposes
the plugin in `/plugins`; selecting it installs the `using-agent-trio` skill.

## Prerequisites

- Codex CLI with `codex plugin marketplace`
- Git

## Installation

1. **Add the agent-trio marketplace:**

   ```bash
   codex plugin marketplace add haplesshero13/agent-trio
   ```

   For local development from a checkout:

   ```bash
   cd /path/to/agent-trio
   codex plugin marketplace add .
   ```

2. **Install from Codex:**

   Restart or open Codex, run `/plugins`, select **Agent Trio**, and install it.

3. **Restart Codex** if the skill is not immediately visible.

## Verify

Ask Codex something that should trigger the workflow, for example:

```text
Use agent-trio to draft a plan for this repo.
```

It should load `using-agent-trio` and create `.trio/plan.md` and
`.trio/criteria.md` for plan-mode requests.

## How it works in Codex

**Plugin marketplace (recommended):**
`.agents/plugins/marketplace.json` exposes the repo-root plugin. Codex reads
`.codex-plugin/plugin.json`, which points at `skills/`, so installing the plugin
from `/plugins` makes `using-agent-trio` available through native Codex plugin
discovery.

**Global skill fallback:**
Older Codex builds, or quick local development setups, can still symlink the
skill directly:

```bash
git clone https://github.com/haplesshero13/agent-trio.git ~/.codex/agent-trio
mkdir -p ~/.codex/skills
ln -sfn ~/.codex/agent-trio/skills/using-agent-trio ~/.codex/skills/using-agent-trio
```

Windows PowerShell fallback:

```powershell
git clone https://github.com/haplesshero13/agent-trio.git "$env:USERPROFILE\.codex\agent-trio"
New-Item -ItemType Directory -Force "$env:USERPROFILE\.codex\skills"
if (Test-Path "$env:USERPROFILE\.codex\skills\using-agent-trio") {
  Remove-Item "$env:USERPROFILE\.codex\skills\using-agent-trio" -Force
}
cmd /c mklink /J "$env:USERPROFILE\.codex\skills\using-agent-trio" "$env:USERPROFILE\.codex\agent-trio\skills\using-agent-trio"
```

**Repo-local agent wrappers (only in a checkout):**
`.codex/agents/builder.toml` and `.codex/agents/reviewer.toml` are thin wrappers that point to `agents/builder.md` and `agents/reviewer.md`. They are only available when Codex runs inside a clone of this repo. If you want to use the builder/reviewer agents in your own project, you must copy **both** the `.toml` wrappers (`from .codex/agents/`) **and** the markdown templates (`from agents/builder.md` and `agents/reviewer.md`) into the corresponding locations in your project. Copying only the `.toml` wrappers is not sufficient — they will fail to load because `agents/builder.md` and `agents/reviewer.md` will not exist.

**Without repo-local agents:**
If builder/reviewer wrappers are not present, Codex acts as builder or reviewer locally, still honoring the full artifact contract (`.trio/plan.md`, `.trio/criteria.md`, `.trio/handoff.md`, `.trio/review.md`).

## Updating

Git-backed marketplace install:

```bash
codex plugin marketplace upgrade agent-trio-marketplace
```

Local checkout marketplace install:

```bash
cd /path/to/agent-trio
git pull
```

Symlink fallback:

```bash
cd ~/.codex/agent-trio
git pull
```

## Uninstalling

Marketplace install: open `/plugins` in Codex and uninstall **Agent Trio**, then
optionally remove the marketplace:

```bash
codex plugin marketplace remove agent-trio-marketplace
```

Symlink fallback:

```bash
rm ~/.codex/skills/using-agent-trio
```

Optionally delete the clone: `rm -rf ~/.codex/agent-trio`
