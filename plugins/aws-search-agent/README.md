# aws-search-agent

A Claude Code plugin bundling the `aws-search` subagent: a policy for sourcing
factual information about AWS — service behavior, features, quotas, pricing,
regional availability, and API/SDK/CloudFormation details — with citations and
freshness markers.

The plugin bundles the official **AWS Knowledge MCP server**
(`https://knowledge-mcp.global.api.aws`, a remote HTTP server, no auth) via
`.mcp.json`. The agent prefers it over a general web search and falls back to
the web only when the MCP server cannot answer.

> Note: a plugin-bundled MCP server loads into the whole session, not just this
> agent — Claude Code does not scope bundled MCP servers per subagent. Enabling
> this plugin connects the AWS Knowledge MCP server for the session.

Use it for AWS-specific questions. For everything else, use `web-search-agent`.

## Install

```bash
/plugin marketplace add 46ki75/claude-plugins
/plugin install aws-search-agent@46ki75-plugins
```
