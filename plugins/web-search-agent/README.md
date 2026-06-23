# web-search-agent

A Claude Code plugin bundling the `web-search` subagent: a policy for sourcing
factual information from the open web — when to look it up, which sources to
trust, and how to report the result with citations and freshness markers.

Use it for fluid questions best answered by a general web search (recent events,
news, prices, current officeholders, trending terminology). It is not for
AWS-specific questions (use `aws-search-agent`), library/framework docs covered
by a dedicated documentation tool, or creative writing.

The agent is restricted to the built-in `WebSearch` and `WebFetch` tools, so the
plugin bundles no MCP server.

## Install

```bash
/plugin marketplace add 46ki75/claude-plugins
/plugin install web-search-agent@46ki75-plugins
```
