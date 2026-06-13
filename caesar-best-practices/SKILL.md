---
name: caesar-best-practices
description: "Reference for implementing Caesar search in code — agentic workflows, RAG systems, autonomous agents, or backend integrations. Use when writing code that calls the Caesar API, SDKs, or CLI, or when choosing between the CLI, MCP server, Python SDK, TypeScript SDK, and raw HTTP."
user-invocable: true
compatibility: Reference material only; no tools required.
license: MIT
metadata:
  author: caesar
---

# Caesar Integration Best Practices

You are implementing a Caesar search integration. Pick the right surface, then
follow the conventions — they are identical across all surfaces.

## Choosing a surface

| Surface | Install | Use when |
|---|---|---|
| CLI `caesar-search` | `npm i -g caesar-search-cli` / brew / curl | Terminal agents, shell scripts, skills. `--json`, typed exit codes 0/2/3/4/5 |
| Python SDK `caesar-search` | `pip install caesar-search` | Python apps. `Caesar`/`AsyncCaesar`, pydantic v2 models |
| TypeScript SDK `caesar-search` | `npm install caesar-search` | Node/Bun/edge apps. ESM+CJS; `caesar-search/ai` exports Vercel AI SDK tools via `caesarTools()` |
| Remote MCP | endpoint `/mcp` on the API host | Chat surfaces (Claude Desktop etc.) that can't run a CLI. Tools: `caesar_search`, `caesar_read` |
| Raw HTTP | OpenAPI spec at `/openapi/public.json` | Everything else. POST `/v1/search`, `/v1/document`, `/v1/feedback` |

## Conventions that hold everywhere

- **Auth**: `Authorization: Bearer $CAESAR_API_KEY`; env var `CAESAR_API_KEY`;
  base URL override `CAESAR_BASE_URL`. Anonymous works at a lower rate limit.
  Never hardcode or log keys.
- **The loop**: search → pick a `doc_id` → read → optionally feedback. Thread
  the provenance handles (`doc_id`, `search_id`, `canonical_url` vs `source_url`,
  crawl/published dates) between calls; they are stable identifiers.
- **Read selection**: `read` without a query returns full-document markdown up to
  the character cap. `read --query` returns a focused selection; if
  `content.truncated` is false, that means the selection fit, not that the full
  document was returned.
- **Response shaping**: `response.verbosity` is `ids_only | compact | standard | full`
  (SDKs/CLI expose it as `verbosity`/`--format`). `compact` is the token-efficient
  choice for agent loops; `full` adds capture provenance. A hard budget is
  available via `response.budget.max_chars_total`.
- **Truncated reads**: the document response reports `content.truncated`,
  `content.start_char`, `content.char_count`. Continue with
  `range.start_char = start_char + char_count` (CLI `--start-char`, SDKs
  `startChar`/`start_char`). Never retry with a bigger character cap.
- **Errors**: every error is an envelope `{"error": {"code", "message", "hint"}}`
  plus `request_id`. Retry 429/5xx with backoff honoring `Retry-After` — the CLI
  and SDKs already do this; raw-HTTP integrations must implement it.
- **Fields are snake_case** exactly as the API returns them, on every surface.
- **Feedback closes the loop**: send `result_helpful` / `stale_result` / etc.
  with `search_id` + `doc_id` when an agent acts on a result; it improves ranking.

## Pitfalls

| Pitfall | Fix |
|---|---|
| Inventing parameter names (`limit`, `n`, `results`) | The parameter is `max_results` (1–50) |
| Expecting camelCase JSON | Everything is snake_case |
| Bare domains to the document endpoint | Full URL or `doc_id` UUID |
| Dropping provenance fields in your data model | Keep them; agents need them to cite and re-fetch |
| Re-requesting a truncated document with a bigger cap | Use the `start_char` continuation |
| Parsing CLI human output | Use `--json`; human output is not a stable interface |

## Live references

- OpenAPI spec: `GET /openapi/public.json` on the API host (codegen-friendly;
  named request/response component schemas)
- CLI: https://github.com/caesar-data/caesar-search-cli (AGENTS.md has the
  common-mistakes table)
- Python SDK: https://github.com/caesar-data/caesar-search-python
- TypeScript SDK: https://github.com/caesar-data/caesar-search-typescript
