# Installing agent-trio for OpenCode

## Prerequisites

- [OpenCode.ai](https://opencode.ai) installed

## Installation

Add agent-trio to the `plugin` array in your `opencode.json` (global or project-level):

```json
{
  "plugin": ["agent-trio@git+https://github.com/haplesshero13/agent-trio.git"]
}
```

Restart OpenCode. The plugin auto-installs and registers all skills and agents.

Verify by asking: "What is the agent-trio workflow?"

## Updating

Agent-trio updates automatically when you restart OpenCode.

To pin a specific version:

```json
{
  "plugin": ["agent-trio@git+https://github.com/haplesshero13/agent-trio.git#v0.1.0"]
}
```

## Troubleshooting

### Plugin not loading

1. Check logs: `opencode run --print-logs "hello" 2>&1 | grep -i trio`
2. Verify the plugin line in your `opencode.json`
3. Make sure you're running a recent version of OpenCode

### Skills not found

1. Use the `skill` tool to list what's discovered
2. Check that the plugin is loading (see above)