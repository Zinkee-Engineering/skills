# Zinkee skills and plugins

Plugins and skills for using [Zinkee](https://zinkee.com) from AI assistants.

## Zinkee plugin (Claude Code and Codex)

[`plugins/zinkee`](plugins/zinkee) connects your assistant to your Zinkee workspace through the
Zinkee MCP server (`https://mcp.zinkee.com/mcp`). You sign in with your own Zinkee user and
choose a workspace; every action uses your permissions.

**Claude Code**

```
/plugin marketplace add Zinkee-Engineering/skills
/plugin install zinkee@zinkee
```

Then `/mcp`, choose `zinkee` and sign in.

**Codex**

```
codex plugin marketplace add Zinkee-Engineering/skills
codex plugin add zinkee@zinkee
```

Claude (claude.ai and desktop), ChatGPT and other assistants can also connect to
`https://mcp.zinkee.com/mcp` directly as a custom connector.

## Skills

- [`skills/zinkee-cli`](skills/zinkee-cli): guide for operating a Zinkee workspace with the
  `zinkee` command-line tool.

## Repository layout

- `.claude-plugin/marketplace.json`: the Claude Code marketplace.
- `.agents/plugins/marketplace.json`: the Codex marketplace.
- `plugins/zinkee`: the plugin, with manifests for both (`.claude-plugin/plugin.json` and
  `plugin.json`).
