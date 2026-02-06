---
name: research-tools
description: External research via Context7 (docs), Grep.app (code examples), and Exa (web search). Loads MCPs on-demand via skill_mcp.
license: MIT
compatibility: opencode
metadata:
  category: research
  triggers: docs, documentation, code examples, web search, how do others, library, API, current info
---

# Research Tools

## Syntax

```
skill_mcp(mcp_name="<SERVER>", tool_name="<TOOL>", arguments='<JSON>')
```

- `mcp_name`: `context7` | `grep_app` | `websearch` (NOT `"research-tools"`)
- `tool_name`: Tool name only (NO server prefix)

## Tools

| Server      | Tool                 | Purpose                     |
| ----------- | -------------------- | --------------------------- |
| `context7`  | `resolve-library-id` | Get library ID (first step) |
| `context7`  | `query-docs`         | Query library docs          |
| `grep_app`  | `searchGitHub`       | GitHub code pattern search  |
| `websearch` | `web_search_exa`     | Web search                  |

## Examples

**Context7** (2-step):

```js
skill_mcp(mcp_name="context7", tool_name="resolve-library-id", arguments='{"libraryName": "react"}')
skill_mcp(mcp_name="context7", tool_name="query-docs", arguments='{"libraryId": "/facebook/react", "query": "useEffect"}')
```

**Grep.app**:

```js
skill_mcp(mcp_name="grep_app", tool_name="searchGitHub", arguments='{"query": "useActionState(", "language": ["TypeScript", "TSX"]}')
```

**Exa**:

```js
skill_mcp(mcp_name="websearch", tool_name="web_search_exa", arguments='{"query": "Next.js 15 features", "numResults": 5}')
```

## Common Mistakes

| Wrong                              | Correct                          |
| ---------------------------------- | -------------------------------- |
| `mcp_name="research-tools"`        | `mcp_name="context7"`            |
| `tool_name="context7_resolve-..."` | `tool_name="resolve-library-id"` |
| `tool_name="grep_app_..."`         | `tool_name="searchGitHub"`       |

## Tool Selection

| Goal                      | Primary     | Secondary  |
| ------------------------- | ----------- | ---------- |
| GitHub issues/discussions | `websearch` | `grep_app` |
| Official docs             | `context7`  | -          |
| Code patterns             | `grep_app`  | -          |
| Current events            | `websearch` | `context7` |
| Library comparison        | `websearch` | `context7` |

Note: For GitHub, prefer MCP tools over `gh` CLI unless auth needed.

## Efficiency Tips

1. **Parallel calls**: Fire independent searches in same message
2. **grep_app**: Search literal code (`"JoinSet<"`), not keywords
3. **context7**: Resolve ID first, skip if cached
4. **Stop when**: same info repeats, 2+ searches yield nothing, or direct answer found

## Output Requirements

- Direct URLs for sources
- Real code snippets (not synthetic)
- Filter for relevance only
- Note dates/versions for current state

## Verification (IMPORTANT)

After this skill loads, you should see an `## Available MCP Servers` section below with:

- `### context7`, `### grep_app`, `### websearch` headers
- `**Tools:**` with `inputSchema` JSON blocks for each

**If missing or shows `*No capabilities discovered*`:**

1. MCP connection failed - retry once with `skill(name="research-tools")`
2. If still missing, warn user: "MCP servers unreachable - research tools unavailable"
3. Fall back to `webfetch` for basic web lookups

**If schemas are present:** Prefer the discovered `inputSchema` over examples in this doc (schemas are authoritative).
