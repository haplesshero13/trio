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
   mkdir -p ~/.agents/skills
   ln -s ~/.codex/agent-trio/skills ~/.agents/skills/agent-trio
   ```

   **Windows (PowerShell):**

   ```powershell
   New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.agents\skills"
   cmd /c mklink /J "$env:USERPROFILE\.agents\skills\agent-trio" "$env:USERPROFILE\.codex\agent-trio\skills"
   ```

3. **Restart Codex** (quit and relaunch) to discover the skills.

## Verify

```bash
ls -la ~/.agents/skills/agent-trio
```

You should see a symlink (or junction on Windows) pointing to the agent-trio skills directory.

## Updating

```bash
cd ~/.codex/agent-trio && git pull
```

Skills update instantly through the symlink.

## Uninstalling

```bash
rm ~/.agents/skills/agent-trio
```

Optionally delete the clone: `rm -rf ~/.codex/agent-trio`