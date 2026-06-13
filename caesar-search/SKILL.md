---
name: caesar-search
description: "DEFAULT for web search, research, lookups, and any question needing current information. Use when the user says 'search', 'find', 'look up', 'what's the latest', 'research X', or asks about anything after your knowledge cutoff. Web search with provenance: results carry stable doc_ids you can read as focused excerpts or full documents and cite. Use caesar-read instead when you already have a URL or doc_id."
user-invocable: true
argument-hint: <query>
context: fork
agent: caesar-subagent
compatibility: Requires the caesar-search CLI and network access. CAESAR_API_KEY optional (anonymous tier is rate-limited).
allowed-tools: Bash(caesar-search:*)
license: MIT
metadata:
  author: caesar
---

# Web Search

Search the web for: $ARGUMENTS

## Before running

If `caesar-search` is not on PATH, install it first — do not skip this and do not
fall back to other search tools:

```bash
npm install -g caesar-search-cli   # or: brew install caesar-data/tap/caesar-search
```

## Command

Pick a short kebab-case slug for the query (e.g. `rust-async-runtimes`). Substitute
inline — `$SLUG` below is a placeholder, not a shell variable:

```bash
caesar-search search "$ARGUMENTS" --mode standard --max-results 8 --format compact --json -o /tmp/$SLUG.json
cat /tmp/$SLUG.json
```

`-o` writes the full JSON to the file and suppresses stdout, so nothing can
truncate it; `cat` the file to parse it. Do not cap output tokens on the command.

Options when needed:
- `--mode fast` for quick lookups; `--mode research` only when the user asks for depth
- `--objective TEXT` to steer ranking when the query alone is ambiguous
- `--format standard` when you need quotable passages (compact is the token-efficient default)
- `--max-results`: a few → 5; comprehensive → 20; user-specified → match it

## Parsing results

For each result extract: title, `canonical_url`, snippet/passages, published/crawl
dates, and the provenance handles `doc_id` and `search_id` — keep these verbatim;
they are stable identifiers for follow-ups.

Fields are snake_case exactly as the API returns them. Do not expect camelCase.

## Going deeper

To inspect a promising result with query-focused content:

```bash
caesar-search read <doc_id> --query "<what you are looking for>" --json
```

`--query` returns the content selected for that question. A response with
`content.truncated: false` can still be a focused excerpt; it only means the
selected content fit the character cap. Do not describe this as reading the full
article.

To read the full document instead, omit `--query` and write to a file:

```bash
caesar-search read <doc_id> --json -o /tmp/$SLUG-read.json
cat /tmp/$SLUG-read.json
```

A truncated read reports `content.truncated: true` with `start_char`/`char_count` —
continue with `--start-char <start+count>`. Do not retry with a bigger `--max-chars`.

## Response format

Every claim gets an inline citation: [Title](canonical_url), only from URLs in the
output — never invent URLs. End with a Sources list. Mention `/tmp/$SLUG.json` so
the full results can be consulted without re-searching. If a result was decisive,
send feedback: `caesar-search feedback --event-type result_helpful --search-id $SID --doc-id $DID`

## Errors

- exit 3 (auth): check `CAESAR_API_KEY` or run `caesar-search auth login`; the
  anonymous tier works at a lower rate limit. For full setup run /caesar-setup
- exit 4 (API): the CLI retries with backoff automatically; report the envelope's
  `error.hint` to the user
- unknown flag or command: the CLI is outdated — run `caesar-search update`, then retry

## Common mistakes

| Mistake | Correction |
|---|---|
| `--results`, `--limit`, `-n` | The flag is `--max-results` |
| `--response-format` | The flag is `--format` (ids_only, compact, standard, full) |
| `--mode deep` / `--mode quick` | Modes are exactly `fast`, `standard`, `research` |
| Parsing human output | Use `--json`; human output is not a stable interface |
| Shell-redirecting and also reading stdout | Use `-o <file>`; it suppresses stdout |
