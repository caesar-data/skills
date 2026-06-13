# Caesar Agent Skills

Agent Skills for Caesar search. These skills teach coding agents the
search -> read -> feedback loop using the `caesar-search` CLI.

Install all skills with the `skills` npm CLI:

```sh
npx skills add caesar-data/skills --skill '*' -g -y
```

Install only the runtime search skill:

```sh
npx skills add caesar-data/skills --skill caesar-search -g -y
```

The skills are:

- `caesar-search` - web search, research, lookups, and current information.
- `caesar-read` - read a URL or `doc_id` as clean markdown with provenance.
- `caesar-setup` - install, authenticate, verify, and update the CLI.
- `caesar-best-practices` - implementation guidance for Caesar integrations.

For Claude Code, the API-hosted installer also writes the bundled
`caesar-subagent` file:

```sh
curl -fsSL https://search-api-staging-779189860552.europe-west1.run.app/skills/install.sh | bash
```

The `npx skills` path installs the `SKILL.md` folders only.
