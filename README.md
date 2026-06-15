# prufa-skill

The Prufa agent skill — 3-tool-call QA quickstart for AI agents. Apache-2.0.

This is the **documentation install surface** for the [prufa-mcp](https://github.com/prufa-dev/prufa-mcp)
server. Point your agent framework at this skill and it learns how to call
Prufa for vibe-coded-app QA.

## Install

For Claude Code / Cursor / Cline / Continue: install the MCP server (the
actual runtime) via `.mcp.json`:

```json
{
  "mcpServers": {
    "prufa": {
      "command": "prufa-mcp",
      "env": { "PRUFA_API_TOKEN": "your-prufa-api-key" }
    }
  }
}
```

Get a free API key at https://prufa.dev — the first audit is free, no card required.

For agent frameworks that read a `SKILL.md` (OpenClaw, Hermes, and similar):
point them at https://github.com/prufa-dev/prufa-skill — the full `SKILL.md`
teaches the agent when to call the 2 free MCP tools, and which hosted tools
exist for the rest of the surface.

## The 2 free tools (the OSS surface)

| Tool | What it does |
|---|---|
| `prufa_run_audit` | One call → runs a public-page audit, returns findings JSON |
| `prufa_get_report` | Fetches a shareable report for a completed audit |

The other ~13 tools (workspace setup, flows, monitors, alerts, billing)
live in the hosted product at https://prufa.dev.

## Why open source

Same shape as [Stagehand](https://github.com/browserbase/stagehand) (free) →
[Browserbase](https://www.browserbase.com) (paid). Open the primitive. The
hosted tier earns the right to be paid by being the thing that scales.

## License

Apache-2.0. See [LICENSE](LICENSE).
