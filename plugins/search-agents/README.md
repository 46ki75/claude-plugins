# search-agents

A Claude Code plugin bundling two research subagents that classify a question as
stable vs. fluid, look information up only when needed, and return cited,
freshness-marked answers:

- **`web-search`** — factual questions best answered by a general web search
  (recent events, news, prices, current officeholders, trending terminology).
  Restricted to the built-in `WebSearch` and `WebFetch` tools, so it bundles no
  MCP server.
- **`aws-search`** — AWS questions (service behavior, features, quotas, pricing,
  regional availability, API/SDK/CloudFormation details). Prefers the bundled
  **AWS Knowledge MCP server** over a general web search, falling back to the web
  only when the MCP server cannot answer.

Claude routes by domain: AWS-specific questions go to `aws-search`, everything
else to `web-search`. Neither covers library/framework docs better served by a
dedicated documentation tool, nor creative writing.

## Bundled MCP server

The plugin bundles the official **AWS Knowledge MCP server**
(`https://knowledge-mcp.global.api.aws`, a remote HTTP server, no auth) via
`.mcp.json`.

> Note: a plugin-bundled MCP server loads into the whole session, not just the
> `aws-search` agent — Claude Code does not scope bundled MCP servers per
> subagent. Enabling this plugin connects the AWS Knowledge MCP server for the
> session.

## Install

```bash
/plugin marketplace add 46ki75/claude-plugins
/plugin install search-agents@46ki75-plugins
```

## Layout

```text
plugins/search-agents/
├── .claude-plugin/plugin.json
├── .mcp.json                 # AWS Knowledge MCP server
├── agents/
│   ├── web-search.md
│   └── aws-search.md
└── evals/
    ├── web-search/           # eval set for the web-search agent
    └── aws-search/           # eval sets for the aws-search agent
```
