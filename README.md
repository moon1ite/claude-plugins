# Claude Code Plugins

Personal plugins for Claude Code.

## Structure

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json      # Plugin metadata (required)
├── agents/              # Agent definitions (optional)
├── skills/              # Skill definitions (optional)
├── commands/            # Slash commands (optional)
└── README.md            # Documentation
```

## Plugins

| Plugin | Description |
|--------|-------------|
| [red-team](./red-team/) | Plan challenger toolkit — dispatches specialized agents to stress-test implementation plans before execution |

## Installation

Install via Claude Code's plugin system:

```
/plugin install red-team@my-plugins
```

Or browse in `/plugin > Discover`.

## Marketplace

This repo is registered as the `my-plugins` marketplace. Add it to your dotfiles via `claude/plugins/known_marketplaces.json`:

```json
{
  "my-plugins": {
    "source": {
      "source": "github",
      "repo": "moon1ite/claude-plugins"
    }
  }
}
```
