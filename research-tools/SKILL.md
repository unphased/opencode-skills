---
name: research-tools
description: Load on-demand MCP research tools with exact `skill_mcp` signatures: Context7 docs, grep_app GitHub search, and Exa web search.
license: MIT
compatibility: opencode
metadata:
  category: research
  triggers: docs, documentation, code examples, web search, how do others, library, API, current info
---

# Research Tools

Use this skill when you need one of these exact MCPs:

- `context7`
- `grep_app`
- `websearch`

## Exact Call Shape

Always call them like this:

```js
skill_mcp(mcp_name="<server>", tool_name="<tool>", arguments='<json>')
```

Rules:
- `mcp_name` must be exactly one of: `context7`, `grep_app`, `websearch`
- `tool_name` is just the tool name, not a prefixed OpenCode tool id
- `arguments` is a JSON string

Do not use:
- `mcp_name="research-tools"`
- `tool_name="context7_resolve-library-id"`
- `tool_name="grep_app_searchGitHub"`
- `tool_name="websearch_web_search_exa"`

## Actual Tools To Use

### `context7`

First call:

```js
skill_mcp(
  mcp_name="context7",
  tool_name="resolve-library-id",
  arguments='{"libraryName":"react"}'
)
```

Then call:

```js
skill_mcp(
  mcp_name="context7",
  tool_name="query-docs",
  arguments='{"libraryId":"/facebook/react","query":"useEffect"}'
)
```

Use `context7` for:
- official docs
- API lookup
- versioned library behavior

### `grep_app`

Use this exact tool:

```js
skill_mcp(
  mcp_name="grep_app",
  tool_name="searchCode",
  arguments='{"query":"useActionState(","langFilter":"TypeScript,TSX","numberedOutput":true}'
)
```

Use `grep_app` for:
- GitHub code patterns
- finding real implementations
- repository-level or language-filtered searches

Other `grep_app` tools that may be useful later:
- `github_file`
- `github_batch_files`
- `batch_retrieve_files`

Search literal code, not vague keywords.
Good: `"JoinSet<"`, `"useActionState("`, `"staleTime:"`
Bad: `"react query example"`

### `websearch`

Use this exact tool:

```js
skill_mcp(
  mcp_name="websearch",
  tool_name="web_search_exa",
  arguments='{"query":"Next.js 15 features","numResults":5}'
)
```

Use `websearch` for:
- current events
- recent releases
- blog posts, issues, discussions
- finding docs URLs when you do not already know the library id

## Fast Tool Selection

- Official docs: `context7`
- Real GitHub code examples: `grep_app`
- Current information or web search: `websearch`
- GitHub issues/discussions: start with `websearch`, then `grep_app` if needed

## Recommended Patterns

### Docs lookup

```js
skill_mcp(mcp_name="context7", tool_name="resolve-library-id", arguments='{"libraryName":"tanstack query"}')
skill_mcp(mcp_name="context7", tool_name="query-docs", arguments='{"libraryId":"/tanstack/query","query":"staleTime gcTime"}')
```

### GitHub code search

```js
skill_mcp(mcp_name="grep_app", tool_name="searchCode", arguments='{"query":"queryOptions(","langFilter":"TypeScript","numberedOutput":true}')
```

### Current/recent info

```js
skill_mcp(mcp_name="websearch", tool_name="web_search_exa", arguments='{"query":"Vite 2026 release notes","numResults":5}')
```

## Efficiency Rules

1. Fire independent searches in parallel when they do not depend on each other.
2. For `context7`, resolve the library id once, then query docs.
3. For `grep_app`, prefer exact syntax fragments over topic words.
4. Stop once two searches are clearly repeating the same information.

## Verification

After loading the skill, check the discovered MCP section:
- `context7`
- `grep_app`
- `websearch`

If capability discovery is missing or empty:
1. Retry `skill(name="research-tools")` once.
2. If still missing, tell the user the MCP servers are unavailable.
3. Fall back to ordinary web lookup tools if needed.

If the discovered schemas disagree with this file, trust the discovered schemas.
