---
name: caesar-read
description: "Read a web page as clean markdown with provenance. Use when you already have a URL or doc_id — 'fetch this page', 'read this URL', 'get the content from', 'what does this article say'. Handles long pages with character-offset continuation. Use caesar-search first when you don't have a URL yet."
user-invocable: true
argument-hint: <url-or-doc_id>
context: fork
agent: caesar-subagent
compatibility: Requires the caesar-search CLI and network access. CAESAR_API_KEY optional (anonymous tier is rate-limited).
allowed-tools: Bash(caesar-search:*)
license: MIT
metadata:
  author: caesar
---

# Read a Page

Read: $ARGUMENTS

## Before running

If `caesar-search` is not on PATH, install it first — do not skip this and do not
fall back to other fetch tools:

```bash
npm install -g caesar-search-cli   # or: brew install caesar-data/tap/caesar-search
```

## Command

The target must be a full URL (`https://…`) or a `doc_id` UUID from caesar-search
results — bare domains are invalid. Pick a short kebab-case slug for the page:

```bash
caesar-search read "$ARGUMENTS" --json -o /tmp/$SLUG.json
cat /tmp/$SLUG.json
```

Options when needed:
- Omit `--query` for full-document markdown (up to `--max-chars`)
- `--query "<question>"` focuses content selection on what you actually need;
  `content.truncated: false` then means the selected excerpt fit, not that the
  full document was returned
- `--max-chars N` caps content size (default 12000)
- The command also answers to `fetch` and `extract`, but write `read` in examples

## Continuing truncated reads

If the response has `content.truncated: true`, continue from where it stopped —
do NOT retry with a bigger `--max-chars`:

```bash
caesar-search read "$ARGUMENTS" --start-char <start_char + char_count> --json -o /tmp/$SLUG-2.json
```

## Response format

Quote or summarize only what the task needs; cite as [Title](canonical_url).
Preserve `doc_id` verbatim for follow-ups and mention the `/tmp/$SLUG.json` path.
Fields are snake_case exactly as the API returns them.

## Errors

- exit 2 with a URL message: the target was probably a bare domain — pass a full URL
- exit 3 (auth): check `CAESAR_API_KEY` or run `caesar-search auth login`; the
  anonymous tier works at a lower rate limit. For full setup run /caesar-setup
- exit 4 (API): the CLI retries with backoff automatically; report `error.hint`
- unknown flag or command: the CLI is outdated — run `caesar-search update`, then retry
