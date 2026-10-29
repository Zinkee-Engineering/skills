# Zinkee

Connects your AI assistant to your [Zinkee](https://zinkee.com) workspace through the Zinkee MCP
server, `https://mcp.zinkee.com/mcp`. It can query and summarise your tables and records, create
and update records, add comments and read the history of a record. Administrators can also create
and change tables and fields.

You sign in with your own Zinkee user and choose one workspace. Everything the assistant does
uses your role and permissions, and is signed by you. You can disconnect it at any time from
**Connected apps** in your Zinkee account menu.

## Install

**Claude Code**

```
/plugin marketplace add Zinkee-Engineering/skills
/plugin install zinkee@zinkee
```

Then run `/mcp`, choose `zinkee` and sign in to Zinkee in the browser that opens.

**Codex**

```
codex plugin marketplace add Zinkee-Engineering/skills
codex plugin add zinkee@zinkee
```

Codex asks you to sign in to Zinkee when you install the plugin or the first time it uses it.
In the `codex` terminal app, `/mcp` shows whether Zinkee is connected.

## What's inside

- `.mcp.json` (Claude Code) and `mcp.json` (Codex and other Agent Plugins hosts): the remote
  Zinkee MCP server, with OAuth sign-in.
- `skills/zinkee`: how to work with Zinkee data safely, such as confirming before deleting.

Privacy policy: <https://zinkee.com/terminos-condiciones#politica-de-privacidad>
