# aws-search eval set

Eval cases for the `aws-search` subagent, run with the
`prompt-evaluation-claude-code` skill (subagents as eval runners — no SDK, no
API key). `eval-set-v1.jsonl` holds 6 cases.

Iteration run artifacts (candidate outputs, judge verdicts) are ephemeral and
kept in the session scratchpad, not committed; only the versioned eval set lives
here.

## Provenance

Two cases are ported from the generalist information-retrieval policy eval set
at `46ki75/prompts` → `prompts/information-retrieval-policy/eval-set-v3.jsonl`
(the NAT Gateway and Bedrock AgentCore cases), reframed from the generalist's
tool-selection wording to "use the AWS Knowledge MCP server." They carry
`"source": "ir-policy-v3:eval-N"`.

Four cases are hand-crafted for this agent (the ported pair alone was too thin
for a viable set) and carry `"source": "new"`:

1. Stable concept — the AWS shared-responsibility model (answer from knowledge,
   no lookup).
2. Quota — default VPCs per Region (fluid; verify via Service Quotas).
3. Regional availability — Bedrock in `ap-northeast-1` (fluid; verify).
4. Security high-stakes — S3 Block Public Access vs. a bucket policy (primary
   AWS docs only).

## Record format

JSONL, one case per line. `id`, `input`, and `criterion` are required; the rest
are optional.

```json
{
  "id": "aws-04",
  "source": "new",
  "input": "What's the default service quota for the number of VPCs per AWS Region?",
  "criterion": "one-sentence pass condition for the binary judge",
  "must_do": ["detailed rubric checklist ..."],
  "must_not_do": ["detailed rubric checklist ..."],
  "tags": ["fluid", "aws", "quota", "mcp-preferred"]
}
```

The candidate subagent runs tool-less, so these grade the agent's **policy and
reasoning** — does it classify stable vs. fluid, prefer the AWS Knowledge MCP,
mark freshness, treat security as high-stakes — not live retrieval.

## Versioning

Bump the filename (`eval-set-v2.jsonl`) when cases change; never mutate a set
that has already been measured against. Stamp every iteration's results with the
eval-set version used. See `prompt-evaluation-claude-code` →
`references/eval_set.md` for the full discipline.
