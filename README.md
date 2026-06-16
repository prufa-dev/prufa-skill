# prufa-skill

The Prufa agent skill — a 2-tool QA quickstart for AI agents. Apache-2.0.

**Which repo do you want?**

- **On Claude Code / Cursor / Cline / Continue?** You don't need this repo —
  just install the MCP server, [prufa-mcp](https://github.com/prufa-dev/prufa-mcp).
  It ships the runtime and the full install guide.
- **On an agent framework that reads a `SKILL.md` (OpenClaw, Hermes, and
  similar)?** This is the repo. Point the framework at [`SKILL.md`](SKILL.md) —
  it teaches the agent *when* to call Prufa, the 2 free MCP tools, the error
  contract, and which hosted tools cover the rest.

## Install the runtime

The skill is documentation; the tools run in the [prufa-mcp](https://github.com/prufa-dev/prufa-mcp)
server. Register it in `.mcp.json` using the **absolute** path to the binary
(`which prufa-mcp` after `pipx install prufa-mcp` — a bare `prufa-mcp` or `~`
does not reliably resolve in MCP host config):

```json
{
  "mcpServers": {
    "prufa": {
      "command": "/Users/you/.local/bin/prufa-mcp",
      "env": { "PRUFA_API_TOKEN": "your-prufa-api-key" }
    }
  }
}
```

Get a free API key at https://prufa.dev — the first audit is free, no card required.
See the [prufa-mcp README](https://github.com/prufa-dev/prufa-mcp) for the
Claude Code `claude mcp add` path and other hosts.

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
