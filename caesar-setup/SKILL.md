---
name: caesar-setup
description: "Install, authenticate, verify, and update the caesar-search CLI. Use when the user wants to set up Caesar search, when caesar-search is missing from PATH, when auth errors (exit 3) persist, or when the CLI needs updating. Conversational lifecycle skill — run it once, then use caesar-search/caesar-read for actual searching."
user-invocable: true
compatibility: Needs network access and one of npm, Homebrew, or curl to install the CLI.
allowed-tools: Bash(command:*), Bash(brew:*), Bash(npm:*), Bash(curl:*), Bash(caesar-search:*)
license: MIT
metadata:
  author: caesar
---

# Caesar Setup

Set up the caesar-search CLI end to end: install → authenticate → verify.

## 1. Detect

```bash
command -v caesar-search && caesar-search version
```

If installed, skip to step 3. If the version is old, run `caesar-search update`
(it detects the install channel — npm, Homebrew, or standalone — and upgrades in
place; `caesar-search update --check --json` reports without changing anything).

## 2. Install

Prefer the self-contained binary channels — they bundle their own runtime, so
nothing on the machine can be too old. Pick the first available, in this order:

```bash
brew install caesar-data/tap/caesar-search
# or
curl -fsSL https://raw.githubusercontent.com/caesar-data/caesar-search-cli/main/install.sh | bash
# or (needs Node >= 22 — the local-render path requires it)
npm install -g caesar-search-cli
```

The curl installer verifies sha256 checksums and lands in `~/.local/bin` (make
sure that is on PATH). The npm package has no postinstall scripts but runs under
the system Node: on Node < 22 reads work but always use the server, so prefer
brew/curl there.

## 3. Authenticate

```bash
caesar-search auth status --json
```

- Search, read, and feedback require an API key. The default fix is
  `caesar-search auth login` — it opens the user's browser and stores a named,
  revocable key (OS keychain, 0600 file fallback). Over SSH or in a container,
  use `caesar-search auth login --device` (short code, approve on any device).
  For CI, ask the user to set `CAESAR_API_KEY`, or store a key with
  `caesar-search auth login --key -` fed from a secret manager. NEVER ask the
  user to paste a key into chat, and never echo keys.
- If the agent runs inside an MCP-capable host (Claude, ChatGPT, Cursor),
  connecting the Caesar connector in the host is an alternative to installing
  the CLI — see the caesar-connect skill.

## 4. Verify

```bash
caesar-search search "hello world" --max-results 1 --format ids_only --json
```

Exit 0 means everything works. Exit 3 → recheck step 3. Exit 4/5 → network or
service issue; the error envelope's `hint` says what to do.

## 5. Suggest first runs

Point the user at the runtime skills: caesar-search (web search) and caesar-read
(read a URL/doc_id as markdown). Example: "search for the latest on X" or
"read https://example.com/post".
